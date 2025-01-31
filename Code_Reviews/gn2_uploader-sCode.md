## The following contains a summary overview of the gn2 uploader source code and how different pieces of the backend architecture are structured and connected. 

### QC_APP-WSGI Script review 
- WSGI, short for "Web Server Gateway Interface", represents an interface between webservers and web applications, in this case python web applications 
- The following is code review for the gn2 uploading app wsgi file
 
#### Section 01: `Module Imports` 
```python 
import os
import sys
from logging import getLogger, StreamHandler
from flask import Flask
from uploader import create_app
from uploader.check_connections import check_db, check_redis
``` 
- `os` and `sys`: interactions with the operating system and system specific parameters 
- `getLogger` and `StreamHandler`: For setting up logging 
- `Flask`: The main class for creating a Flask web application
- `create_app`: A function from the uploader module to initialize the Flask app
- `check_db` and `check_redis`: Functions to verify database and Redis connections

#### Section 02: Logging setup function
 
```python 
def setup_logging(appl: Flask) -> Flask:
    """Setup appropriate logging paradigm depending on environment."""
```
- The function above is defined to configure loggings for the application based on the environment 

```python 
    software, *_version_and_comments = os.environ.get(
        "SERVER_SOFTWARE", "").split('/')
    if bool(software):
        gunicorn_logger = getLogger("gunicorn.error")
        appl.logger.handlers = gunicorn_logger.handlers
        appl.logger.setLevel(gunicorn_logger.level)

```
- The above piece of code checks if the application is running under a specific server (Gunicorn in this case)
- If so, it uses the server's configurations 

```python 
    else:
        loglevel = appl.config["LOG_LEVEL"].upper()
        handler_stderr = StreamHandler(stream=sys.stderr)
        appl.logger.addHandler(handler_stderr)
        rootlogger = getLogger()
        rootlogger.addHandler(handler_stderr)
        rootlogger.setLevel(loglevel)
        appl.logger.setLevel(loglevel)

```
- The above piece of code is for setting up the default loggings, in case the app does not run on Gunicorn
  - It retrieves the log level from the app's configuration 
  - Creates a stream handler that outputs logs to standard error (stderr) 
  - Configures both, the application logger and root logger to use the handler and log level 

```python 
    return appl
```
- Returns the application with already configured logging setup 

#### Section 03: Application Setup function
 
```python 
def check_and_build_app() -> Flask:
    """Setup the application for running."""
```
- The defined function above is used to create and configure the app (Flask app) 

```python 
    appl = create_app()
    check_db(appl.config["SQL_URI"])
    check_redis(appl.config["REDIS_URL"])
    return setup_logging(appl)
```
- The piece of code above does the following; 
    - `create_app()` instantiate the Flask app 
    - Then checks for database connectivity using SQL URI from the app configuration 
    - Then checks for redis connectivity using the Redis URL from app configuration 
    - Finally, it sets up logging for the app before returning it 

```python 
app = check_and_build_app()
```
- The function to create and configure the app is called and assigned to a variable `app` 

#### Section 04: Running the app 

```python 
if __name__ == "__main__":
    # Run the app
    app.run()

```
- The code above checks if the script is being run directly (i.e, not imported as a module)
- If so, it starts the Flask development server to run the application 

#### In summary, this scripts sets up a Flask Web application with proper logging and checks for necessary connections to databases and services before running it 

