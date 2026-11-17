
package com.connecthub.app

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

private val Blue = Color(0xFF1457B8)
private val LightBlue = Color(0xFFEAF2FF)
private val Dark = Color(0xFF17243A)

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                ConnectHubApp()
            }
        }
    }
}

@Composable
fun ConnectHubApp() {
    var page by remember { mutableStateOf("Home") }
    var role by remember { mutableStateOf("Student") }
    var name by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var loggedIn by remember { mutableStateOf(false) }

    if (!loggedIn) {
        LoginScreen(
            name = name,
            email = email,
            onNameChange = { name = it },
            onEmailChange = { email = it },
            onLogin = { loggedIn = true }
        )
        return
    }

    Scaffold(
        containerColor = Color(0xFFF5F8FE),
        bottomBar = {
            NavigationBar(containerColor = Color.White) {
                listOf("Home", "Lectures", "Notices", "Profile").forEach {
                    NavigationBarItem(
                        selected = page == it,
                        onClick = { page = it },
                        icon = {},
                        label = { Text(it) }
                    )
                }
            }
        }
    ) { padding ->
        Column(
            Modifier.fillMaxSize().padding(padding)
        ) {
            Header()

            when (page) {
                "Home" -> HomeScreen(name, role) {
                    page = it
                }
                "Lectures" -> LecturesScreen(role)
                "Notices" -> NoticesScreen()
                "Profile" -> ProfileScreen(
                    name, email, role,
                    onRoleChange = { role = it },
                    onLogout = { loggedIn = false }
                )
                "Report" -> ReportScreen()
                "Support" -> SupportScreen()
            }
        }
    }
}

@Composable
fun LoginScreen(
    name: String,
    email: String,
    onNameChange: (String) -> Unit,
    onEmailChange: (String) -> Unit,
    onLogin: () -> Unit
) {
    var selectedRole by remember { mutableStateOf("Student") }

    Column(
        Modifier.fillMaxSize()
            .background(Color.White)
            .padding(25.dp),
        verticalArrangement = Arrangement.Center
    ) {
        Text("ConnectHub", fontSize = 34.sp,
            fontWeight = FontWeight.Bold, color = Blue)

        Text("Your university. Connected.",
            color = Dark, fontSize = 17.sp)

        Spacer(Modifier.height(35.dp))

        OutlinedTextField(
            value = name,
            onValueChange = onNameChange,
            label = { Text("Full name") },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(Modifier.height(12.dp))

        OutlinedTextField(
            value = email,
            onValueChange = onEmailChange,
            label = { Text("Email address") },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(Modifier.height(15.dp))

        Text("Continue as", fontWeight = FontWeight.Bold)

        Row {
            listOf("Student", "Lecturer").forEach {
                Row(
                    Modifier.weight(1f),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    RadioButton(
                        selected = selectedRole == it,
                        onClick = { selectedRole = it }
                    )
                    Text(it)
                }
            }
        }

        Button(
            onClick = onLogin,
            enabled = name.isNotBlank() &&
                    email.contains("@"),
            modifier = Modifier.fillMaxWidth(),
            colors = ButtonDefaults.buttonColors(
                containerColor = Blue
            )
        ) {
            Text("Continue")
        }

        Text(
            "Prototype: account verification and secure authentication will be connected in the next development stage.",
            fontSize = 12.sp,
            color = Color.Gray
        )
    }
}

@Composable
fun Header() {
    Column(
        Modifier.fillMaxWidth()
            .background(Blue)
            .padding(20.dp)
    ) {
        Text("CONNECTHUB",
            color = Color.White,
            fontSize = 23.sp,
            fontWeight = FontWeight.Bold)
        Text("Learn. Connect. Succeed.",
            color = Color.White)
    }
}

@Composable
fun HomeScreen(name: String, role: String, navigate: (String) -> Unit) {
    LazyColumn(
        Modifier.fillMaxSize().padding(16.dp)
    ) {
        item {
            Text("Welcome, $name 👋",
                fontSize = 23.sp,
                fontWeight = FontWeight.Bold,
                color = Dark)
            Text("$role Dashboard", color = Blue)
            Spacer(Modifier.height(20.dp))
        }

        item {
            DashboardCard("📚", "My Lectures",
                "Access your university lecture rooms") {
                navigate("Lectures")
            }
        }
        item {
            DashboardCard("📢", "Announcements",
                "Read important university updates") {
                navigate("Notices")
            }
        }
        item {
            DashboardCard("🛡️", "Confidential Reporting",
                "Report suspected misconduct securely") {
                navigate("Report")
            }
        }
        item {
            DashboardCard("💬", "Help & Support",
                "Get assistance with ConnectHub") {
                navigate("Support")
            }
        }
    }
}

@Composable
fun DashboardCard(
    emoji: String,
    title: String,
    description: String,
    onClick: () -> Unit
) {
    Card(
        Modifier.fillMaxWidth()
            .padding(vertical = 7.dp)
            .clickable { onClick() },
        colors = CardDefaults.cardColors(
            containerColor = Color.White
        ),
        shape = RoundedCornerShape(16.dp)
    ) {
        Column(Modifier.padding(18.dp)) {
            Text("$emoji  $title",
                fontSize = 18.sp,
                fontWeight = FontWeight.Bold,
                color = Blue)
            Spacer(Modifier.height(5.dp))
            Text(description, color = Dark)
        }
    }
}

@Composable
fun LecturesScreen(role: String) {
    var showCreate by remember { mutableStateOf(false) }
    var roomName by remember { mutableStateOf("") }
    var courseCode by remember { mutableStateOf("") }
    var joinCode by remember { mutableStateOf("") }
    var message by remember { mutableStateOf("") }

    Column(Modifier.fillMaxSize().padding(18.dp)) {
        Text("Lecture Rooms",
            fontSize = 25.sp,
            fontWeight = FontWeight.Bold,
            color = Dark)

        Spacer(Modifier.height(15.dp))

        if (role == "Lecturer") {
            Button(onClick = { showCreate = !showCreate }) {
                Text("Create Lecture Room")
            }
        } else {
            OutlinedTextField(
                value = joinCode,
                onValueChange = { joinCode = it },
                label = { Text("Enter invitation code") },
                modifier = Modifier.fillMaxWidth()
            )
            Button(
                onClick = {
                    message = if (joinCode.isNotBlank())
                        "Join request prepared for $joinCode"
                    else "Enter an invitation code"
                }
            ) {
                Text("Join Lecture")
            }
        }

        if (showCreate && role == "Lecturer") {
            OutlinedTextField(
                value = roomName,
                onValueChange = { roomName = it },
                label = { Text("Lecture title") },
                modifier = Modifier.fillMaxWidth()
            )
            OutlinedTextField(
                value = courseCode,
                onValueChange = { courseCode = it },
                label = { Text("Course code") },
                modifier = Modifier.fillMaxWidth()
            )
            Button(onClick = {
                message = if (roomName.isNotBlank() &&
                    courseCode.isNotBlank())
                    "Room details saved locally: $roomName"
                else "Complete all fields"
            }) {
                Text("Save Room")
            }
        }

        Spacer(Modifier.height(15.dp))
        Text(message, color = Blue)
        Text("Your lecture rooms will appear here.",
            color = Color.Gray)
    }
}

@Composable
fun NoticesScreen() {
    Column(Modifier.padding(20.dp)) {
        Text("University Notices",
            fontSize = 25.sp,
            fontWeight = FontWeight.Bold)
        Spacer(Modifier.height(15.dp))
        DashboardCard("📅", "Academic Calendar",
            "Academic dates and important events") {}
        DashboardCard("📢", "University Updates",
            "Official announcements") {}
    }
}

@Composable
fun ProfileScreen(
    name: String,
    email: String,
    role: String,
    onRoleChange: (String) -> Unit,
    onLogout: () -> Unit
) {
    Column(Modifier.padding(20.dp)) {
        Text("My Profile", fontSize = 25.sp,
            fontWeight = FontWeight.Bold)
        Spacer(Modifier.height(15.dp))
        Text("Name: $name")
        Text("Email: $email")
        Text("Current role: $role")
        Spacer(Modifier.height(15.dp))
        Text("Prototype role selector")
        Button(onClick = {
            onRoleChange(
                if (role == "Student") "Lecturer"
                else "Student"
            )
        }) {
            Text("Switch demo role")
        }
        Spacer(Modifier.height(15.dp))
        OutlinedButton(onClick = onLogout) {
            Text("Log out")
        }
    }
}

@Composable
fun ReportScreen() {
    var anonymous by remember { mutableStateOf(true) }
    var report by remember { mutableStateOf("") }
    var message by remember { mutableStateOf("") }

    Column(Modifier.padding(20.dp)) {
        Text("Confidential Reporting",
            fontSize = 24.sp,
            fontWeight = FontWeight.Bold)
        Text("Report suspected bribery or misconduct.")

        Row(verticalAlignment = Alignment.CenterVertically) {
            Checkbox(anonymous, { anonymous = it })
            Text("Submit anonymously")
        }

        OutlinedTextField(
            value = report,
            onValueChange = { report = it },
            label = { Text("Describe your report") },
            modifier = Modifier.fillMaxWidth(),
            minLines = 4
        )

        Button(onClick = {
            message = if (report.isNotBlank())
                "Report prepared. Secure submission is not yet connected."
            else "Please enter your report."
        }) {
            Text("Prepare Report")
        }

        Text(message, color = Blue)
    }
}

@Composable
fun SupportScreen() {
    Column(Modifier.padding(20.dp)) {
        Text("Help & Support",
            fontSize = 25.sp,
            fontWeight = FontWeight.Bold)
        Spacer(Modifier.height(12.dp))
        Text("ConnectHub support")
        Text("Automated assistance: 24/7")
        Text("Human assistance: University working hours")
        Spacer(Modifier.height(15.dp))
        Text("Frequently Asked Questions")
        Text("• How do I join a lecture?")
        Text("• How do lecturers create rooms?")
        Text("• How do I access lecture recordings?")
        Text("• How do I report misconduct?")
    }
}
# CONNECT-HUB