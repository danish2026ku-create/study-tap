package com.yourname.focusstudy // Change this to your actual package name at the top of the file

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.CheckCircle
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Timer
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.StrokeCap
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextDecoration
import androidx.compose.ui.unit.dp
import com.yourname.focusstudy.ui.theme.FocusStudyTheme // Keep your default theme import

// --- DATA CLASS ---
data class StudyTask(val id: Int, val title: String, var isCompleted: Boolean = false)

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            FocusStudyTheme {
                Surface(color = MaterialTheme.colorScheme.background) {
                    StudyApp()
                }
            }
        }
    }
}

@Composable
fun StudyApp() {
    // --- STATE MANAGEMENT ---
    var currentScreen by remember { mutableStateOf("timer") }

    // Timer State
    val totalTime = 25 * 60 * 1000L // 25 minutes
    var timeLeft by remember { mutableStateOf(totalTime) }
    var isRunning by remember { mutableStateOf(false) }

    // Tasks State
    val tasks = remember { mutableStateListOf<StudyTask>() }
    var newTaskText by remember { mutableStateOf("") }
    var nextTaskId by remember { mutableIntStateOf(1) }

    // --- TIMER LOGIC ---
    LaunchedEffect(isRunning) {
        if (!isRunning) return@LaunchedEffect

        while (isRunning && timeLeft > 0L) {
            kotlinx.coroutines.delay(1000L)
            timeLeft -= 1000L
        }

        if (timeLeft <= 0L) {
            timeLeft = 0L
            isRunning = false
        }
    }

    // --- UI LAYOUT ---
    Scaffold(
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    icon = { Icon(Icons.Default.Timer, contentDescription = "Timer") },
                    label = { Text("Focus") },
                    selected = currentScreen == "timer",
                    onClick = { currentScreen = "timer" }
                )
                NavigationBarItem(
                    icon = { Icon(Icons.Default.CheckCircle, contentDescription = "Tasks") },
                    label = { Text("Tasks") },
                    selected = currentScreen == "tasks",
                    onClick = { currentScreen = "tasks" }
                )
            }
        }
    ) { innerPadding ->
        Box(modifier = Modifier.padding(innerPadding).fillMaxSize()) {
            if (currentScreen == "timer") {
                TimerScreen(
                    timeLeft = timeLeft,
                    totalTime = totalTime,
                    isRunning = isRunning,
                    onStartPause = { isRunning = !isRunning },
                    onReset = {
                        isRunning = false
                        timeLeft = totalTime
                    }
                )
            } else {
                TasksScreen(
                    tasks = tasks,
                    newTaskText = newTaskText,
                    onTextChange = { newTaskText = it },
                    onAddTask = {
                        val taskTitle = newTaskText.trim()
                        if (taskTitle.isNotEmpty()) {
                            tasks.add(StudyTask(id = nextTaskId, title = taskTitle))
                            nextTaskId += 1
                            newTaskText = ""
                        }
                    },
                    onToggleTask = { id ->
                        val index = tasks.indexOfFirst { it.id == id }
                        if (index != -1) {
                            val task = tasks[index]
                            tasks[index] = task.copy(isCompleted = !task.isCompleted)
                        }
                    },
                    onDeleteTask = { id ->
                        tasks.removeAll { it.id == id }
                    }
                )
            }
        }
    }
}

// --- TIMER SCREEN UI ---
@Composable
fun TimerScreen(
    timeLeft: Long,
    totalTime: Long,
    isRunning: Boolean,
    onStartPause: () -> Unit,
    onReset: () -> Unit
) {
    val progress = if (totalTime > 0L) {
        ((totalTime - timeLeft).toFloat() / totalTime.toFloat()).coerceIn(0f, 1f)
    } else {
        0f
    }
    val minutes = (timeLeft / 1000L) / 60L
    val seconds = (timeLeft / 1000L) % 60L
    val timeString = String.format("%02d:%02d", minutes, seconds)

    Column(
        modifier = Modifier.fillMaxSize().padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Focus Session",
            style = MaterialTheme.typography.headlineMedium,
            fontWeight = FontWeight.Bold
        )

        Spacer(modifier = Modifier.height(48.dp))

        // Circular Progress
        Box(contentAlignment = Alignment.Center, modifier = Modifier.size(250.dp)) {
            CircularProgressIndicator(
                progress = { progress },
                modifier = Modifier.fillMaxSize().clip(CircleShape),
                strokeWidth = 16.dp,
                trackColor = MaterialTheme.colorScheme.surfaceVariant,
                strokeCap = StrokeCap.Round
            )
            Text(
                text = timeString,
                style = MaterialTheme.typography.displayLarge,
                fontWeight = FontWeight.Bold
            )
        }

        Spacer(modifier = Modifier.height(48.dp))

        // Controls
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            Button(
                onClick = onStartPause,
                modifier = Modifier.weight(1f).height(60.dp).padding(end = 8.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = if (isRunning) MaterialTheme.colorScheme.error else MaterialTheme.colorScheme.primary
                )
            ) {
                Text(if (isRunning) "Pause" else "Start", style = MaterialTheme.typography.titleMedium)
            }

            OutlinedButton(
                onClick = onReset,
                modifier = Modifier.weight(1f).height(60.dp).padding(start = 8.dp)
            ) {
                Text("Reset", style = MaterialTheme.typography.titleMedium)
            }
        }
    }
}

// --- TASKS SCREEN UI ---
@Composable
fun TasksScreen(
    tasks: List<StudyTask>,
    newTaskText: String,
    onTextChange: (String) -> Unit,
    onAddTask: () -> Unit,
    onToggleTask: (Int) -> Unit,
    onDeleteTask: (Int) -> Unit
) {
    Column(modifier = Modifier.fillMaxSize().padding(16.dp)) {
        Text(
            text = "My Tasks",
            style = MaterialTheme.typography.headlineMedium,
            fontWeight = FontWeight.Bold,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        // Add Task Input
        Row(
            modifier = Modifier.fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically
        ) {
            OutlinedTextField(
                value = newTaskText,
                onValueChange = onTextChange,
                label = { Text("New Task") },
                modifier = Modifier.weight(1f),
                singleLine = true
            )
            Spacer(modifier = Modifier.width(8.dp))
            IconButton(
                onClick = onAddTask,
                modifier = Modifier.size(56.dp)
            ) {
                Icon(Icons.Default.Add, contentDescription = "Add Task", tint = MaterialTheme.colorScheme.primary)
            }
        }

        Spacer(modifier = Modifier.height(16.dp))

        // Task List
        LazyColumn(verticalArrangement = Arrangement.spacedBy(8.dp)) {
            items(tasks, key = { it.id }) { task ->
                Card(
                    modifier = Modifier.fillMaxWidth(),
                    colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surfaceVariant)
                ) {
                    Row(
                        modifier = Modifier.padding(12.dp),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Checkbox(
                            checked = task.isCompleted,
                            onCheckedChange = { onToggleTask(task.id) }
                        )

                        Text(
                            text = task.title,
                            modifier = Modifier.weight(1f).padding(horizontal = 8.dp),
                            style = MaterialTheme.typography.bodyLarge,
                            textDecoration = if (task.isCompleted) TextDecoration.LineThrough else TextDecoration.None,
                            color = if (task.isCompleted) MaterialTheme.colorScheme.onSurfaceVariant else MaterialTheme.colorScheme.onSurface
                        )

                        IconButton(onClick = { onDeleteTask(task.id) }) {
                            Icon(Icons.Default.Delete, contentDescription = "Delete", tint = MaterialTheme.colorScheme.error)
                        }
                    }
                }
            }
        }
    }
}
