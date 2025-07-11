# Application Usage Guide: Livebook Task Scheduler

This guide explains how to use the Livebook Task Scheduler to define, schedule, and monitor simple tasks.

## 1. Overview

The Task Scheduler is a Livebook application that allows you to define tasks and schedule them to run at specific times or intervals, similar to cron jobs on a Linux system. You interact with it directly within your Livebook environment.

*(Note: The main `README.md` file for this project includes a screenshot (`assets/task_manager.png`) that gives a general visual idea of the application. This usage guide focuses on the interactive elements as generated by the `task_scheduler.livemd` notebook.)*

Key components you'll interact with:
*   **Input Form:** Used to define and add new tasks.
*   **Scheduled Tasks Table:** Displays the list of tasks you've added.
*   **Livebook Output:** Shows messages when tasks "execute".

## 2. Setup and Starting the Scheduler

Before you can schedule tasks, you need to initialize the application within your Livebook notebook (`task_scheduler.livemd`):

1.  **Open the Notebook:** Launch Livebook and open the `task_scheduler.livemd` file.
2.  **Install Dependencies:** Run the first Elixir code cell that contains `Mix.install([...])`. This will download and install `kino` and `crontab` if they are not already available.
    ```elixir
    Mix.install([
      {:kino, "~> 0.16.0"},
      {:crontab, "~> 1.1"}
    ])
    ```
3.  **Define Core Components:** Execute the cell containing the `TaskStore` and `TaskRunner` module definitions.
4.  **Start the Application & UI:** Run the cells under the "Application UI" section. This will:
    *   Start the `TaskStore` and `TaskRunner` processes.
    *   Render the input form for adding tasks.
    *   Prepare the frame where scheduled tasks will be displayed.
    ```elixir
    # --- UI Frames ---
    tasks_frame = Kino.Frame.new()

    # --- Start Application ---
    {:ok, _} = TaskStore.start_link([])
    {:ok, _} = TaskRunner.start_link(tasks_frame: tasks_frame)

    # --- Initial Render ---
    Process.sleep(100)
    send(Process.whereis(TaskRunner), :render_ui)

    # --- Input Form ---
    form =
      Kino.Control.form(
        [
          name: Kino.Input.text("Task Name"),
          cron_string: Kino.Input.text("Cron String")
        ],
        submit: "Add Task"
      )
    ```
5.  **Enable Task Submission:** Execute the cell under the "Task Submission Handler" section. This allows the "Add Task" button to function.
    ```elixir
    Kino.listen(form, fn %{data: %{name: name, cron_string: cron_string}} ->
      # ...
    end)
    ```
6.  **Display Task Table:** Finally, execute the cell under "Scheduled Tasks" that contains `tasks_frame`. This will show the initially empty table where your tasks will appear.

After these steps, the Task Scheduler is ready to use.

## 3. Adding a New Scheduled Task

To schedule a new task:

1.  **Locate the Input Form:** You should see a form with two fields: "Task Name" and "Cron String", and an "Add Task" button.
2.  **Enter Task Name:** In the "Task Name" field, type a descriptive name for your task (e.g., "Daily Report Generator", "Backup Script").
3.  **Enter Cron String:** In the "Cron String" field, enter a cron expression that defines when the task should run.
    *   **What is a Cron String?** A cron string is a sequence of fields that specify the timing for a recurring task. The format typically is: `minute hour day_of_month month day_of_week`.
    *   **Example:** `*/10 * * * * *` means "every 10 seconds". (Note: This scheduler uses a 6-field cron string including seconds). `0 9 * * 1` would mean "at 09:00 on every Monday".
    *   For help constructing cron strings, you can use online tools like [crontab.guru](https://crontab.guru/).
4.  **Submit the Task:** Click the "Add Task" button.

The task will be added to the system, and the "Scheduled Tasks" table will update to include your new task.

## 4. Viewing Scheduled Tasks

Once you've added tasks, they will appear in the "Scheduled Tasks" table below the input form. The table displays the following information for each task:

*   **Task Name:** The name you provided for the task.
*   **Cron String:** The schedule you defined for the task.
*   **Status:** This will typically show as `:pending`. Even when a task "executes," its status in this table does not change in the current version.

## 5. Monitoring Task Execution

When a task's scheduled time arrives, the `TaskRunner` will "execute" it. In the current version of the scheduler, this means a message will be printed to the Livebook's standard output/log area.

*   **Look for Output:** Keep an eye on the output section of your Livebook notebook. You will see messages like:
    ```
    Executing task: Your Task Name
    ```
    This indicates that the task was due and the system attempted to run it.

## 6. Current Limitations

Please be aware of the following limitations in the current version of the Livebook Task Scheduler:

*   **No Task Editing:** Once a task is added, you cannot modify its name or cron string through the UI.
*   **No Task Deletion:** There is no UI option to remove a task from the schedule. To clear tasks, you would typically need to restart the `TaskStore` Agent, which usually means re-running the relevant Livebook cells or restarting the Livebook kernel for that section.
*   **Status Display:** The "Status" column in the tasks table always shows `:pending` and does not update to reflect execution.
*   **Simple Execution:** Task "execution" is currently simulated by printing a message to the console. No actual external commands or complex Elixir functions are run.

This usage guide should help you get started with the Livebook Task Scheduler. As it's a Livebook application, you can also explore the code cells directly to understand its workings in more detail.
