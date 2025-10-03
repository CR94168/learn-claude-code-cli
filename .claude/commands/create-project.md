# Create Project with framework and name. argument-hint: [framework-name] [project-name]

You are a **software architect and code generator**.  
Your task is to **scaffold a new starter project** inside a folder with the given project name, using the specified framework.

## Arguments
1. `{{args[0]}}` = framework to use (e.g., "ASP.NET Core Web API", "Node.js Express REST API", "React + Vite frontend", "Django web app").
2. `{{args[1]}}` = project name → the folder under the current root where the project should be created.

## Instructions
1. Parse framework: `{{args[0]}}`.  
2. Parse project folder name: `{{args[1]}}`.  
3. Create a new folder named `{{args[1]}}` at the project root.  
4. Scaffold a **starter project** that matches the framework:
   - Correct folder layout inside `{{args[1]}}/`
   - Dependency/configuration files (`package.json`, `.csproj`, `requirements.txt`, etc.)
   - Basic entry point (`Program.cs`, `server.js`, `main.py`, `main.tsx`, etc.)
   - Example route/component if applicable  
5. Follow framework conventions exactly (do not invent your own).  
6. Show:
   - Project folder tree
   - File contents for initial setup
7. Never overwrite unrelated files outside `{{args[1]}}`.

⚠️ Keep starter minimal but functional. Only include framework-required files.
