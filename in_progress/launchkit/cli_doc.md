# LaunchKit CLI Documentation

## Overview
LaunchKit is a CLI tool for creating full-functioning Rails 8 or Laravel 11 projects based on user input. It leverages AI to generate an initial database model tailored to customer needs. Projects created with LaunchKit are ready to be deployed using your preferred stack and database.

---

## Command Reference

### `launchkit help`
Displays information about available commands and their usage.

Example:
```bash
$ launchkit help
Available commands:
  authenticate      Authenticate with your API token.
  create            Create a new project.
  re-authenticate   Re-enter your API token.
  version           Show the current version of LaunchKit.
  help              Show this help message.
```

---

### `launchkit authenticate`
Authenticates the user with an API token.

Example:
```bash
$ launchkit authenticate
Please enter your API token:
[Token input hidden]
Authentication successful! You can now create projects with LaunchKit.
```

---

### `launchkit re-authenticate`
Re-authenticates the user if their token has expired or changed.

Example:
```bash
$ launchkit re-authenticate
Please enter your new API token:
[Token input hidden]
Re-authentication successful!
```

---

### `launchkit create`
Starts the project creation flow.

Example:
```bash
$ launchkit create
Welcome to LaunchKit! Let’s create your new project.
```

Follow the steps below during the `create` flow.

---

### `launchkit version`
Displays the current version of the CLI.

Example:
```bash
$ launchkit version
LaunchKit CLI version 1.0.0
```

---

## Project Creation Flow

When you run `launchkit create`, you will be guided through the following steps:

1. **Introduction**
   ```bash
   Welcome to LaunchKit! Let’s create your new project.
   ```

2. **Project Name**
   ```bash
   What is the name of your project?
   > [User Input]
   ```

3. **Project Description**
   ```bash
   Describe your project in a few sentences. This helps us tailor the database and features to your needs.
   > [User Input]
   ```

4. **Technology Stack**
   ```bash
   Which technology stack do you prefer?
   1. Rails 8
   2. Laravel 11
   > [User Input (1 or 2)]
   ```

5. **Database**
   ```bash
   Which database do you prefer for your project?
   1. PostgreSQL
   2. SQLite
   3. MySQL
   > [User Input (1, 2, or 3)]
   ```

6. **Team Support**
   ```bash
   Does your project need support for teams? (e.g., multiple users working together in the app)
   [yes/no]
   > [User Input]
   ```

7. **Confirmation**
   ```bash
   Here’s a summary of your project:
   - Name: [Project Name]
   - Description: [Project Description]
   - Stack: [Rails 8 or Laravel 11]
   - Database: [PostgreSQL/SQLite/MySQL]
   - Team Support: [Yes/No]

   Does this look correct? (yes/no)
   > [User Input]
   ```
   - If "no": The CLI will prompt for corrections and loop back through questions.
   - If "yes": The process continues.

8. **Submission**
   ```bash
   Sending your project details to LaunchKit...
   Your project is being generated. This may take a few moments.
   ```

9. **Completion**
   ```bash
   Success! Your project has been created.
   You can find your new project in the current directory. Run the following commands to get started:

   cd [project-name]
   [stack-specific instructions, e.g., bundle install & rails s for Rails]

   Thank you for using LaunchKit!
   ```

---

## Notes and Enhancements
1. **Interactive Help**: Each question includes brief context for better decision-making.
2. **Default Values**: Defaults are suggested based on common preferences or prior use.
3. **Customization**: Support additional project features or configurations based on user inputs.
4. **Logging**: Log project details for debugging or analytics (optional).

---

## Version
Current Version: `1.0.0`

