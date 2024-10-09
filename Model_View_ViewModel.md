# Model_View_ViewModel
Model_View_ViewModel
````
class ComposeClass {

    @Composable
    fun TaskScreen(viewModel: TaskViewModel = viewModel()) {
        val tasks by viewModel.tasks.collectAsState()
        var taskName by remember { mutableStateOf("") }

        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            verticalArrangement = Arrangement.Top,
            horizontalAlignment = Alignment.Start
        ) {
            // Input field for new task
            BasicTextField(
                value = taskName,
                onValueChange = { taskName = it },
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(8.dp)
                    .border(1.dp, MaterialTheme.colorScheme.primary)
                    .padding(8.dp),
                decorationBox = { innerTextField ->
                    if (taskName.isEmpty()) {
                        Text("Enter task name")
                    }
                    innerTextField()
                }
            )

            Spacer(modifier = Modifier.height(8.dp))

            Button(onClick = {
                if (taskName.isNotBlank()) {
                    viewModel.addTask(taskName)
                    taskName = ""
                }
            }) {
                Text("Add Task")
            }

            Spacer(modifier = Modifier.height(16.dp))

            // Displaying the list of tasks
            LazyColumn {
                items(tasks) { task ->
                    TaskItem(task, viewModel)
                }
            }
        }
    }

    @Composable
    fun TaskItem(task: TaskViewModel.Task, viewModel: TaskViewModel) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Checkbox(
                checked = task.isCompleted,
                onCheckedChange = {
                    viewModel.toggleTaskCompletion(task)
                }
            )
            Text(
                text = task.name,
                style = MaterialTheme.typography.bodyMedium,
                modifier = Modifier.padding(start = 8.dp)
            )
        }
    }

}
````
````
class TaskViewModel : ViewModel() {

    // StateFlow to hold the list of tasks
    private val _tasks = MutableStateFlow<List<Task>>(emptyList())
    val tasks: StateFlow<List<Task>> = _tasks

    // Function to add a task
    fun addTask(taskName: String) {
        val newTask = Task(id = _tasks.value.size + 1, name = taskName)
        viewModelScope.launch {
            _tasks.value = _tasks.value + newTask
        }
    }

    // Function to toggle task completion
    fun toggleTaskCompletion(task: Task) {
        viewModelScope.launch {
            _tasks.value = _tasks.value.map {
                if (it.id == task.id) it.copy(isCompleted = !it.isCompleted) else it
            }
        }
    }

    data class Task(
        val id: Int,
        val name: String,
        var isCompleted: Boolean = false
    )

}
````
````
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Model_View_ViewModelTheme {

                ComposeClass().TaskScreen()

                }
            }
        }
    }
````
