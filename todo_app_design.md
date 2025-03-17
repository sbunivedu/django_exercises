# To-Do App Design

## **Learning Objective**
Students will learn how to design a Django-based **To-Do App** by first defining **use case scenarios**, sketching a **low-fi UI design**, and deriving the **functional requirements** before implementing the solution.

## **Step 1: Understanding the Core Use Cases**
Before writing any code, let's think about **who** will use the app and **what they need to do**.

### **Use Case 1: User Registration & Authentication**
*Scenario:* A user wants to sign up, log in, and log out.

**Requirements:**
- Allow users to register with a username and password.
- Users should log in to access their tasks.
- Provide a logout button.

### **Use Case 2: Creating a Task**
*Scenario:* A logged-in user wants to add a new task to their to-do list.

**Requirements:**
- Display a form with a task title and optional description.
- Save the task to the database.
- Show the newly created task on the dashboard.

### **Use Case 3: Editing a Task**
*Scenario:* A user wants to modify an existing task.

**Requirements:**
- Provide an "Edit" button for each task.
- Pre-fill a form with the task’s current details.
- Allow users to save updates.

### **Use Case 4: Marking a Task as Complete**
*Scenario:* A user wants to mark a task as completed.

**Requirements:**
- Provide a checkbox or button to mark a task as complete.
- Update the UI to indicate the task is done.

### **Use Case 5: Deleting a Task**
*Scenario:* A user wants to remove a task they no longer need.

**Requirements:**
- Show a "Delete" button for each task.
- Ask for confirmation before deletion.
- Remove the task from the list upon confirmation.

### **Use Case 6: Undoing a Deletion (Using Sessions)**
*Scenario:* A user accidentally deletes a task and wants to undo it.

**Requirements:**
- Store the recently deleted task in the session.
- Provide an "Undo" button that restores the task.
- Remove the task from the session after a few minutes.

## **Step 2: Designing the Low-Fi UI Sketches**
Here are simple **low-fidelity wireframes** to visualize the design.

### **Login & Registration Page**
```
+--------------------------+
|      To-Do App           |
+--------------------------+
| Username: [__________]   |
| Password: [__________]   |
| [ Login ]  [ Register ]  |
+--------------------------+
```

### **To-Do List Dashboard**
```
+---------------------------------+
| Welcome, [User]                 |
|                                 |
| [ New Task ]                    |
+---------------------------------+
| Task 1          [ Edit ] [✔] [X]|
| Task 2          [ Edit ] [✔] [X]|
| Task 3          [ Edit ] [✔] [X]|
+---------------------------------+
| [ Logout ]                      |
| [ Undo Last Delete ] (if needed)|
+---------------------------------+
```

### **Task Creation / Editing Form**
```
+---------------------------+
|  [ Task Name:  ______ ]   |
|  [ Description: _____ ]   |
|                           |
|  [ Save Task ]            |
+---------------------------+
```

## **Step 3: Deriving the Functional Requirements**

From the above **use cases** and **sketches**, we define the core **Django functionalities**:

### **User Authentication**
- Django’s built-in authentication system (`django.contrib.auth`)
- `User` model for managing logins

### **Task Model (Database)**
- `Task` model with fields: `title`, `description`, `is_completed`, `created_at`

### **Views and URLs**
- `login_view()`, `logout_view()`, `register_view()`, `task_list_view()`, `task_create_view()`, `task_edit_view()`, `task_delete_view()`, `task_undo_view()`

### **Forms**
- Django Forms to handle task creation and editing

### **Sessions for Undo Feature**
- Store deleted tasks in Django sessions

## **Step 4: Hands-On Implementation**
Now that students understand the functional design, they can start implementing the app. They should:

1. **Set up the Django project** (`django-admin startproject todo_app`)
2. **Create the `Task` model** with the required fields
3. **Implement user authentication** with Django’s `auth` module
4. **Build views and templates** using the lo-fi sketches
5. **Use Django sessions** to implement the "Undo Delete" feature
6. **Style the app** using Bootstrap

## **Discussion Questions**
1. How do the **use cases** help define the app’s functionality?
2. Why are **low-fidelity sketches** useful before writing code?
3. What are some challenges of implementing the **Undo Delete** feature?
4. How can we improve the user experience of this to-do app?

## **Suggested Learning Resources**
- **Django Authentication**: [Django Docs - User Authentication](https://docs.djangoproject.com/en/stable/topics/auth/)
- **Django Forms**: [Django Docs - Forms](https://docs.djangoproject.com/en/stable/topics/forms/)
- **Django Sessions**: [Django Docs - Sessions](https://docs.djangoproject.com/en/stable/topics/http/sessions/)
