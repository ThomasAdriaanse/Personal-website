title: How this website works
date: 2023-01-02
published: true

### Introduction

I wanted to showcase my full-stack development skills with a personal website. I started by choosing a simple template from [this GitHub repo](https://github.com/buildwithdan/flask-portfolio) that would allow me to easily add my own touch.

![Website Diagram](/static/images/website_diagram.png)
### Flask

Flask is a lightweight web framework that allowed me to get up and running quickly without unnecessary complexity. One of its key strengths is minimalist design—since Flask doesn’t enforce a particular structure, I can customize it to my needs. Also, Flask is widely used in the Python community, and I am familiar with Python, making it an easy choice for the backend.

##### Why Flask Over Django?

While Django provides more built-in features, Flask’s simplicity gave me more control over what I wanted to implement without the overhead of a larger framework. For a personal project like this, where I didn’t need a full-blown admin interface or built-in user authentication, Flask seemed a more efficient option.

### AWS EC2

I chose AWS EC2 to host my website, managing the server manually without auto-scaling or clustering for simplicity and because I don't expect much traffic. Since the traffic for this project is likely low, EC2 with Docker is a cost-effective solution. AWS Fargate offers automatic scaling and simpler container management, but it costs more and i thought it was a good idea to more hands on experience with AWS instances. Besides, EC2 offers more control over the server environment if i want to do fancy stuff in the future.

### Docker

I like docker because it eliminates the typical "works on my machine" problem via containerisation. With Docker, I package all the dependencies, including the Flask app and the web server, into a container which ensures my development and production environments are identical.

For example, I used a Docker image with a pre-configured Python environment and dependencies, which allows me to test locally before pushing the image to AWS. Using Docker in production also helps with easier updates. Instead of manually installing or updating dependencies, I simply rebuild the Docker image, push it to my Elastic Container Registry (ECR), and pull it down on the EC2 instance.

### Nginx

I’m using Nginx as a reverse proxy to route web traffic to my Flask app. Nginx is known for its high performance and reliability in serving static content and handling multiple client requests efficiently. It allows me to offload some of the heavy lifting, such as managing incoming HTTP requests, which improves the scalability and security of my website.

#### Benefits of Nginx

Nginx adds a layer of security to my stack by acting as a gateway that routes traffic to the Flask app. It also supports SSL termination, which helps to encrypt traffic between the client and server. With Nginx, I can configure caching and optimize request handling, ensuring that the server remains responsive even under load.

### Gunicorn

Initially, I was exposing Flask’s default development server on port 5000, which isn’t suitable for production as it’s neither optimized nor secure. Enter Gunicorn—a WSGI server designed for running Python web applications in production.

#### Benefits of Gunicorn

Gunicorn acts as a bridge between Nginx and my Flask application, handling multiple client requests by spawning worker processes. It’s fast, reliable, and easily integrates with Flask. By using Gunicorn in conjunction with Nginx, I ensure that my app can handle concurrent traffic while maintaining security.

### SSL Encryption

To ensure secure communication between users and my site, I used SSL/TLS certificates, obtained from Porkbun. This protects data in transit, giving visitors confidence that their interactions with my site are secure.