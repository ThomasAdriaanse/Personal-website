title: The Devops Revelation
date: 2023-01-02
published: yes


I started doing Capture The Flags just for fun a while back because computer security is pretty interesting, and who doesn't want to feel like a hacker. The problem with offensive computer security is that it can be very overwhelming to try and attack somthing if you dont have a very good understanding of what you are trying to attack. A lot of the CTFs I would do were on web based applications, so I decided a great way to get better at attacking them was to make one myself. The other reason I wanted to make a website was to have a convienent place to put stuff I make so that people could easily check it out.

Because of my noob status in building web applications, I stareted with a template (template from [this GitHub repo](https://github.com/buildwithdan/flask-portfolio)) and decided to work from there. I really didnt change the internal structure of the website much, like the flask routing and the frontend design, but I made a second site later where I focused on learning those areas.

I was mainly concerned with learning about the infrastructure which enables the site to run properly, so thats what this post will be about, the devops. Here is a diagram of the devops pipeline for the website:

![Website Diagram](/static/images/website_diagram.png)

I'll try to break it down in parts and explain what ive learned.

I used Amazon Web Service (AWS) because I figured it would be the most useful to learn long-term considering its popularity. AWS has a bunch of services, most of which seem to be in service of their primary product: cloud compute. Their cloud compute is called Elastic Cloud Compute or EC2 for short. You can rent an EC2 server to use for about 15$ CAD per month on the cheapest end, which is what I am using (note: "server" is an incredibly broad term for a machine which performs a service. This is an appropriate term for EC2s because that's what they do: preform any service you want from the cloud). When you rent an EC2 instead of it being a phycisal machine, you rent an "instance" which is a virtual machine, which is a part of a larger computer. You can interact with your EC2 instance by SSHing into it like you would any other computer. From there you can pull code from a repository, like Amazon's Elastic Container Registry. Because I want my code to be viewable to the public and because i'm used to it, I just used Github as the repoistory I push to, but I push to Amazon ECR as well. From there when I ssh into my EC2, I can pull the new code from my ECR and restart the web service using the newly pulled code.

I used Nginx and gunicorn to host my web server. I use gunicorn for multiple processes for better fault tolerance and for parallelism. For the Nginx conf file I just copied an online template and added the correct path to my SSL certificates and made sure to serve the site over http and https, redirecting any http traffic to https. The only annoying thing about using https is that I have to manually grab the new SSL certificated bundle every few months, but ill probably automate that later. I use Porkbun as my domain and SSL provider, but I am not sure if they have an API for grabbing the bundles. Seems unlikely given that they require an authenticator app for logins but you never know.

