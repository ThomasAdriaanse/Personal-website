title: Devops
date: 2024-09-11
published: No




# Learning Devops
Previously when I was creating my website, I tried to be as simple as possible for the developmment pipeline. I figuered that I prefer to spend my time learning about the infrastruccture of websites, then the infrastructure of the development process. My process looked like this:

1. Set up github repo only using main
2. Write code on VS code, using the integrated git source control UI
3. Push code to Github when I wanted to see the code on Dev build
4. IGNORE the giithub repo and push my changes to my amazon ecr repo 
5. Log in to my amazon EC2 instance and pull the latest build from the ecr
6. deleted old doker build, buld using new image, restart nginx.

This is pretty terrible for a number of reasons, which I realized, so i have decided to make a proper devops system using Github Actions.

