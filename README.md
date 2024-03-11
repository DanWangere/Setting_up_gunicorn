# Setting Up Gunicorn for Apache Superset
To deploy Superset in a production environment, it's recommended to use **Gunicorn** as the application server.

## What is Gunicorn?

**Gunicorn** (short for **Green Unicorn**) is a Python Web Server Gateway Interface (WSGI) HTTP server. It serves as a bridge between your Python application (such as Apache Superset) and the web server (such as Apache or Nginx). Gunicorn manages incoming requests, handles concurrency, and ensures smooth communication between your application and the outside world.

## Why Use Gunicorn with Apache Superset?

1. **Performance**: Gunicorn is designed for high performance and can handle multiple requests concurrently.
2. **Stability**: It's a reliable choice for serving Python applications in production.
3. **Scalability**: Gunicorn can be easily scaled horizontally by adding more worker processes.
4. **Integration**: When combined with a web server like Apache or Nginx, Gunicorn provides a robust setup for deploying Superset.

In the following sections, we'll walk through the steps to set up Gunicorn for Apache Superset.

1. **Install Gunicorn**:
   First, ensure you have Python and pip installed. Then, install Gunicorn using pip:
   ```bash
   pip install gunicorn
   ```
   Install the following too:
   ```bash
    pip install greenlet                # Required for both
    pip install eventlet                # For eventlet workers
    pip install gunicorn[eventlet]      # Or, using extra
    pip install gevent                   # For gevent workers
    pip install gunicorn[gevent]         # Or, using extra 
   ```

2. **Create a Gunicorn Configuration File**:
   Create a file named `gunicorn_config.py` (or any other name you prefer) with the following content:
   ```python
   # gunicorn_config.py

   # Number of worker processes (adjust based on your server's resources)
   gunicorn -w 10 \

   # Bind to a specific address and port (e.g., localhost:8000)
   -k gevent \
      --worker-connections 1000 \
      --timeout 120 \
      -b  0.0.0.0:8088 \
      --limit-request-line 0 \
      --limit-request-field_size 0 \

   # Set Superset's WSGI application
   "superset.app:create_app()"
   ```

3. **Run Gunicorn**:
   Navigate to your Superset installation directory and run Gunicorn with the configuration file:
   ```bash
   gunicorn -c gunicorn_config.py
   ```

4. **Set up WSGI**:

     ```bash
     /superset/lib/python3.8/site-packages/superset$ sudo nano wsgi.py
     ```

   - Create a wsgi file :
     ```apache
     from superset.app import create_app
     app = create_app()

     if __name__ == "__main__":
     app.run()

  - run to see if it’s working :
    
    ```bash 
    gunicorn -b 0.0.0.0:8088 superset.wsgi:app

     ```
   - Create a Service:
     ```bash
     cd /etc/systemd/system
     sudo nano superset.service

     ```
- Paste the following:
    ```bash
    
    [Unit]
    Description=Superset service
    After=network.target

    [Service]
    Type=simple
    User=<username>
    WorkingDirectory=/home/<username>
    Environment="FLASK_APP=superset"
    #ExecStart=/home/<username>/venv/bin/gunicorn --log-level debug -w 4 -b <portnumber>:8088 superset.wsgi:app
    #ExecStart=/home/<username>/superset/bin/superset run -h <portnumber>  -p 8088 --with-threads --reload
    #ExecStart=/home/<username>/superset/bin/gunicorn  -c superset/lib/python3.10/site-packages/superset/gunicorn_config.py superset/lib/python3.10/site-packages/s>
    ExecStart=/home/<username>/superset/bin/gunicorn -b <portnumber>:8088 superset.wsgi:app
    Restart=always

    [Install]
    WantedBy=multi-user.target

    ```

5. **Save and Run the following**:

   ```bash
    systemctl daemon-reload
    sudo systemctl enable superset.service
    sudo systemctl start superset.service

   ```

6. **Access Superset**:
   Open your web browser and visit `http://your-domain.com` (replace with your actual domain). You should see the Superset login page.

And there you have it! You've set up Gunicorn to serve Apache Superset.  🚀