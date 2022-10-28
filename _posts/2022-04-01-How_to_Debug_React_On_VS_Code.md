---
layout: post
title: "How to Debug React on VS Code"
date: 2022-04-01
excerpt: "How to Debug React on VS Code"
tags: ["Debugging", "React", "VSCode"]
comments: true
---


1. Make launch.json right under root directory
    
   <img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled.png">
    
    ```java
    {
        "version": "0.2.0",
        "configurations": [
            {
                "type": "chrome",
                "request": "launch",
                "name": "Launch Chrome against localhost",
                "url": "http://localhost:3000",
                "webRoot": "${workspaceFolder}/src",
                "remoteRoot": "D:\\personal\\IdeaProjects\\wherehaveubeen\\react-social\\.vscode\\launch.json",
    
            }
        ]
    }
    ```
    
2. start project on terminal
    
    ```java
    npm start
    ```
    
   <img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%201.png">
    

1. makes some breakpoints 
    
   <img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%202.png">
    

1. On Run and Debug tab, launch configuration. 
    
    Select Play Button. 
    
   <img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%203.png">