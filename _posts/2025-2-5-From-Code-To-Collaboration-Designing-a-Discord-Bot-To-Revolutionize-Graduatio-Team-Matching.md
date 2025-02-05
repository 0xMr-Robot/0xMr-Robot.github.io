---
layout: post
title: "From Code to Collaboration: Designing a Discord Bot to Revolutionize Graduation Team Matching"
date: 2025-02-05 12:00:00 -0400
categories: 
  - bot-development
  - discord-bots
  - student-tools
  - automation
tags: 
  - discord
  - bot
  - team-matching
  - graduation
  - automation
  - university
  - teamwork
  - student-collaboration
author: "Eslam Abbas"
image: /assets/img/posts/2025-2-5-From-Code-To-Collaboration-Designing-a-Discord-Bot-To-Revolutionize-Graduatio-Team-Matching/Discord_Bot.png 
---


# From Code to Collaboration: Designing a Discord Bot to Revolutionize Graduation Team Matching 

In the world of university projects, team formation can often be a daunting task. Coordinating skills, interests, and schedules among a diverse group of students is no easy feat. Enter the **Graduation Team Matching Discord Bot**, a powerful tool designed to automate and simplify the process of forming teams for graduation projects. This bot leverages cutting-edge technology to match students based on their academic background, technical skills, and personal preferences, creating the perfect team dynamic in just a few clicks. In this post, we’ll explore how this bot was developed, its key features, and how it can transform the way students collaborate on their final projects.


## 📝 Summary of the Bot

The **Graduation Team Matching Discord Bot** is a powerful tool designed to streamline the process of forming teams for graduation projects. It automates the matching of **team leaders** and **members** based on their skills, interests, and project requirements. By leveraging a sophisticated algorithm, the bot ensures that teams are formed efficiently, saving time and reducing the hassle of manual coordination.

This bot is particularly useful for university students who need to collaborate on graduation projects. It categorizes users into **leaders** and **members**, allows them to select their preferred **tracks** (e.g., Backend Frameworks, Cybersecurity, Data Science), and matches them based on their **technical skills**, **university**, and **department**. The bot also provides a detailed profile of each user, including their **selected topics**, **ratings**, and **comments**, to ensure compatibility.

---

## 🌟 Key Features

1. **Role-Based Registration**:
   - Users can register as either a **Team Leader** or a **Team Member**.
   - Leaders can create teams and specify project ideas, while members can showcase their skills and interests.

2. **Track Selection**:
   - Users can choose from a wide range of **tracks** such as:
     - Backend Frameworks (e.g., .NET, Node.js, Django)
     - Cybersecurity (e.g., Network Security, Ethical Hacking)
     - Data Science (e.g., Machine Learning, Data Analysis)
     - And more!

3. **Topic-Based Matching**:
   - Members can select topics they’ve studied within their chosen track.
   - The bot calculates a **rating** based on the difficulty and relevance of the selected topics.

4. **University and Department Matching**:
   - Ensures that leaders and members are from the same **university** and **department** for better collaboration.

5. **Automated Matching Algorithm**:
   - Uses a **priority queue** to match members with leaders based on their ratings and registration time.
   - Sends **direct messages** to both parties with match details.

6. **Data Persistence**:
   - Saves user data (e.g., registrations, matches) to a JSON file for persistence across bot restarts.

7. **User-Friendly Commands**:
   - Simple commands like `!start`, `!choose_track`, and `!match` make the bot easy to use.

8. **Error Handling and Logging**:
   - Comprehensive error handling and logging ensure smooth operation and easy debugging.

---

## 💡 Benefits of Using the Bot

- **Efficiency**: Automates the team formation process, saving time and effort.
- **Fairness**: Matches are based on **ratings** and **registration time**, ensuring a fair process.
- **Compatibility**: Ensures that team members and leaders are aligned in terms of skills, interests, and academic background.
- **Transparency**: Provides detailed match information to both leaders and members.
- **Scalability**: Can handle a large number of users across multiple tracks and universities.

---

## 🛠️ Installation Guide

To install and run the **Graduation Team Matching Discord Bot**, follow these steps:

### Prerequisites
1. **Python 3.8 or higher**: Ensure Python is installed on your system.
2. **Discord Bot Token**: Create a bot on the [Discord Developer Portal](https://discord.com/developers/applications) and obtain the bot token.
3. **Git**: Install Git to clone the repository.

### Step 1: Clone the Repository
Open your terminal or command prompt and run:
```bash
git clone https://github.com/0xMr-Robot/Graduation_Team_Matching_Discord_Bot.git
cd Graduation_Team_Matching_Discord_Bot
```

### Step 2: Install Dependencies
Install the necessary dependencies using pip:

```bash
pip install -r requirements.txt
```
This command ensures that all the required Python libraries are installed. The dependencies listed in the requirements.txt file include the libraries the bot relies on, such as:

    discord.py: The main library for creating the Discord bot.
    python-dotenv: Used to load environment variables from a .env file.
    schedule: For scheduling tasks like automatic bot restarts.

### Step 3: Create a `.env` File
In the root of the project, create a `.env` file to store your Discord Bot token and other sensitive information.

```text
DISCORD_BOT_TOKEN=your_discord_bot_token_here
```
This .env file is essential for securely storing your bot's token and any other private credentials. The DISCORD_BOT_TOKEN is what allows your bot to authenticate with Discord's servers. You’ll need to replace your_discord_bot_token_here with the actual bot token you obtained when creating your bot on the [Discord Developer Portal](https://discord.com/developers/applications).

The python-dotenv library will automatically load the environment variables from this file when the bot starts running.

### Step 4: Run the Bot
After completing the setup, run the bot with the following command:

```bash
python bot.py
```
This command will start the bot, and it will begin listening for interactions on your Discord server. The bot will connect to Discord's API, and you’ll see a message in your terminal indicating that the bot has logged in successfully.

Once the bot is running, it will be able to respond to commands like `!start`, `!choose_track`, and others, as described in the bot's functionality.
If you need guidance on how to use the bot effectively, feel free to watch this video tutorial that walks you through the process:
[Watch the video tutorial here](https://youtu.be/F2J-7enuEAs)


If you encounter any errors or issues while running the bot, check the logs or error messages that appear in your terminal. If necessary, review the steps above to ensure everything is set up correctly.


## 💡 Detailed Explanation of the Code

Now that we have the bot set up, let’s break down the core components of the bot's code in detail. We’ll go through the most important parts that help the bot perform its task effectively — from setting up logging to user registration and the matching algorithm.

### 💻 The Full Code 
This is the full code of my Discord Bot which will be explaind into points now : 

```python
import json
import logging
from datetime import datetime
import schedule
import signal
from logging.handlers import RotatingFileHandler
import discord
from discord.ext import commands, tasks
from discord.ext.commands import cooldown, BucketType
from discord.ui import Button, View, Select
from collections import defaultdict
import heapq
import asyncio
import os
import sys
from dotenv import load_dotenv
import time

# Setup logging
log_formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
log_file = 'bot_logs.log'
log_handler = RotatingFileHandler(log_file, maxBytes=5*1024*1024, backupCount=5)
log_handler.setFormatter(log_formatter)

logger = logging.getLogger('BotLogger')
logger.setLevel(logging.INFO)
logger.addHandler(log_handler)



# Load environment variables from .env file
load_dotenv()

# Initialize the bot
intents = discord.Intents.default()
intents.message_content = True
intents.members = True
bot = commands.Bot(command_prefix="!", intents=intents , heartbeat_timeout=120)


# Categorized Tracks
track_categories = {
    "Backend Frameworks": [".net", "node.js", "laravel", "django", "spring"],
    "Cybersecurity Specializations": ["network security", "ethical hacking", "digital forensics"],
    "Data Science Specializations": ["machine learning", "data analysis", "data engineering", "deep learning"],
    "Other Tracks": ["front end", "ui-ux", "flutter", "cloud", "mobile", "embedded systems", "vr", "game development"]
}

universities = [
    "Cairo University",
    "Ain Shams University",
    "Alexandria University",
    "Helwan University",
    "Mansoura University",
    "Assiut University",
    "Zagazig University",
    "Tanta University",
    "Suez Canal University",
    "Benha University",
    "Fayoum University",
    "South Valley University",
    "Menoufia University",
    "Port Said University",
    "Beni Suef University",
    "Kafrelsheikh University",
    "Damietta University",
    "Sohag University",
    "Modern Academy",
    "MSA University",
    "MTI University",
    "Future University",
    "October 6 University",
    "Badr University",
    "New Cairo Academy"
]

# Track Topics with Difficulty Scores
track_topics = {
    ".net": [
        {"name": "C# Basics", "difficulty": "beginner", "score": 15},
        {"name": ".NET Core Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Web API", "difficulty": "intermediate", "score": 25},
        {"name": ".NET MVC", "difficulty": "intermediate", "score": 25},
        {"name": "Entity Framework", "difficulty": "advanced", "score": 40},
        {"name": "Dependency Injection", "difficulty": "advanced", "score": 40},
        {"name": "Microservices with .NET", "difficulty": "advanced", "score": 40},
        {"name": "Advanced ORM", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "node.js": [
        {"name": "JavaScript Basics", "difficulty": "beginner", "score": 15},
        {"name": "Node.js Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Express.js Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Async Programming", "difficulty": "intermediate", "score": 25},
        {"name": "RESTful API Design", "difficulty": "advanced", "score": 40},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Express", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Tuning", "difficulty": "advanced", "score": 40}
    ],
    "laravel": [
        {"name": "PHP Basics", "difficulty": "beginner", "score": 15},
        {"name": "Laravel Installation", "difficulty": "beginner", "score": 15},
        {"name": "Routing Fundamentals", "difficulty": "intermediate", "score": 25},
        {"name": "Eloquent ORM", "difficulty": "intermediate", "score": 25},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "Blade Templates", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Eloquent", "difficulty": "advanced", "score": 40},
        {"name": "Laravel Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "django": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Django Setup", "difficulty": "beginner", "score": 15},
        {"name": "Basic Models", "difficulty": "intermediate", "score": 25},
        {"name": "Django ORM", "difficulty": "intermediate", "score": 25},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "REST Framework", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Querying", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "spring": [
        {"name": "Java Basics", "difficulty": "beginner", "score": 15},
        {"name": "Spring Boot Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Dependency Injection", "difficulty": "intermediate", "score": 25},
        {"name": "Spring MVC", "difficulty": "intermediate", "score": 25},
        {"name": "JPA", "difficulty": "advanced", "score": 40},
        {"name": "Security Configuration", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Caching", "difficulty": "advanced", "score": 40},
        {"name": "Performance Tuning", "difficulty": "advanced", "score": 40}
    ],
    "network security": [
        {"name": "Network Basics", "difficulty": "beginner", "score": 15},
        {"name": "TCP/IP", "difficulty": "beginner", "score": 15},
        {"name": "Firewall Concepts", "difficulty": "intermediate", "score": 25},
        {"name": "Intrusion Detection", "difficulty": "intermediate", "score": 25},
        {"name": "Network Protocols", "difficulty": "advanced", "score": 40},
        {"name": "Packet Analysis", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Threat Detection", "difficulty": "advanced", "score": 40},
        {"name": "Network Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Secure Network Design", "difficulty": "advanced", "score": 40}
    ],
    "ethical hacking": [
        {"name": "Security Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Networking", "difficulty": "beginner", "score": 15},
        {"name": "Linux Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Penetration Testing Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Vulnerability Assessment", "difficulty": "advanced", "score": 40},
        {"name": "Exploit Techniques", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Penetration Testing", "difficulty": "advanced", "score": 40},
        {"name": "Red Team Tactics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Exploit Development", "difficulty": "advanced", "score": 40}
    ],
    "digital forensics": [
        {"name": "Computer Forensics Basics", "difficulty": "beginner", "score": 15},
        {"name": "Evidence Preservation", "difficulty": "beginner", "score": 15},
        {"name": "Basic Tools", "difficulty": "intermediate", "score": 25},
        {"name": "Forensic Analysis Techniques", "difficulty": "intermediate", "score": 25},
        {"name": "Disk Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Memory Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Forensic Tools", "difficulty": "advanced", "score": 40},
        {"name": "Malware Analysis", "difficulty": "advanced", "score": 40},
        {"name": "Complex Investigation Techniques", "difficulty": "advanced", "score": 40}
    ],
    "machine learning": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Statistics Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic ML Algorithms", "difficulty": "intermediate", "score": 25},
        {"name": "Scikit-learn", "difficulty": "intermediate", "score": 25},
        {"name": "Supervised Learning", "difficulty": "advanced", "score": 40},
        {"name": "Feature Engineering", "difficulty": "advanced", "score": 40},
        {"name": "Deep Learning", "difficulty": "advanced", "score": 40},
        {"name": "Advanced ML Algorithms", "difficulty": "advanced", "score": 40},
        {"name": "Model Deployment", "difficulty": "advanced", "score": 40}
    ],
    "data analysis": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Excel Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Statistics", "difficulty": "intermediate", "score": 25},
        {"name": "Pandas", "difficulty": "intermediate", "score": 25},
        {"name": "NumPy", "difficulty": "advanced", "score": 40},
        {"name": "Data Visualization", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Analytics", "difficulty": "advanced", "score": 40},
        {"name": "Predictive Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Big Data Tools", "difficulty": "advanced", "score": 40}
    ],
    "data engineering": [
        {"name": "SQL Basics", "difficulty": "beginner", "score": 15},
        {"name": "Data Warehousing Concepts", "difficulty": "beginner", "score": 15},
        {"name": "ETL Fundamentals", "difficulty": "intermediate", "score": 25},
        {"name": "Apache Spark", "difficulty": "intermediate", "score": 25},
        {"name": "Big Data Technologies", "difficulty": "advanced", "score": 40},
        {"name": "Data Pipeline Design", "difficulty": "advanced", "score": 40},
        {"name": "Distributed Computing", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Data Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Real-time Data Processing", "difficulty": "advanced", "score": 40}
    ],
    "deep learning": [
        {"name": "Neural Network Basics", "difficulty": "beginner", "score": 15},
        {"name": "Python for AI", "difficulty": "beginner", "score": 15},
        {"name": "Basic Deep Learning Concepts", "difficulty": "intermediate", "score": 25},
        {"name": "TensorFlow", "difficulty": "intermediate", "score": 25},
        {"name": "Keras", "difficulty": "advanced", "score": 40},
        {"name": "Neural Network Architectures", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Neural Networks", "difficulty": "advanced", "score": 40},
        {"name": "Computer Vision", "difficulty": "advanced", "score": 40},
        {"name": "NLP Techniques", "difficulty": "advanced", "score": 40}
    ],
    "front end": [
        {"name": "HTML Basics", "difficulty": "beginner", "score": 15},
        {"name": "CSS Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "JavaScript Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Responsive Design", "difficulty": "intermediate", "score": 25},
        {"name": "Bootstrap", "difficulty": "advanced", "score": 40},
        {"name": "JavaScript ES6+", "difficulty": "advanced", "score": 40},
        {"name": "React", "difficulty": "advanced", "score": 40},
        {"name": "Vue.js", "difficulty": "advanced", "score": 40},
        {"name": "State Management", "difficulty": "advanced", "score": 40}
    ],
    "ui-ux": [
        {"name": "Design Principles", "difficulty": "beginner", "score": 15},
        {"name": "Color Theory", "difficulty": "beginner", "score": 15},
        {"name": "Typography Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Wireframing", "difficulty": "intermediate", "score": 25},
        {"name": "Prototyping", "difficulty": "advanced", "score": 40},
        {"name": "User Research", "difficulty": "advanced", "score": 40},
        {"name": "Figma", "difficulty": "advanced", "score": 40},
        {"name": "Adobe XD", "difficulty": "advanced", "score": 40},
        {"name": "Advanced UX Strategy", "difficulty": "advanced", "score": 40}
    ],
    "flutter": [
        {"name": "Dart Basics", "difficulty": "beginner", "score": 15},
        {"name": "Flutter Installation", "difficulty": "beginner", "score": 15},
        {"name": "Basic Widgets", "difficulty": "intermediate", "score": 25},
        {"name": "State Management", "difficulty": "intermediate", "score": 25},
        {"name": "Navigation", "difficulty": "advanced", "score": 40},
        {"name": "API Integration", "difficulty": "advanced", "score": 40},
        {"name": "Custom Widgets", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40},
        {"name": "Advanced State Solutions", "difficulty": "advanced", "score": 40}
    ],
    "cloud": [
        {"name": "Cloud Computing Basics", "difficulty": "beginner", "score": 15},
        {"name": "Basic Networking", "difficulty": "beginner", "score": 15},
        {"name": "Virtual Machines", "difficulty": "intermediate", "score": 25},
        {"name": "AWS Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Azure Fundamentals", "difficulty": "advanced", "score": 40},
        {"name": "Docker Basics", "difficulty": "advanced", "score": 40},
        {"name": "Kubernetes", "difficulty": "advanced", "score": 40},
        {"name": "Cloud Security", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Deployment Strategies", "difficulty": "advanced", "score": 40}
    ],
    "mobile": [
        {"name": "Mobile Development Basics", "difficulty": "beginner", "score": 15},
        {"name": "UI Design for Mobile", "difficulty": "beginner", "score": 15},
        {"name": "Android Development", "difficulty": "intermediate", "score": 25},
        {"name": "iOS Development", "difficulty": "intermediate", "score": 25},
        {"name": "React Native Basics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Mobile Architectures", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40},
        {"name": "Cross-Platform Development", "difficulty": "advanced", "score": 40}
    ],
    "embedded systems": [
        {"name": "Electronics Basics", "difficulty": "beginner", "score": 15},
        {"name": "Microcontroller Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "C Programming", "difficulty": "intermediate", "score": 25},
        {"name": "Arduino Programming", "difficulty": "intermediate", "score": 25},
        {"name": "Raspberry Pi", "difficulty": "advanced", "score": 40},
        {"name": "Sensor Integration", "difficulty": "advanced", "score": 40},
        {"name": "IoT Protocols", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Embedded Programming", "difficulty":"advanced", "score": 40},
        {"name": "Real-Time Systems", "difficulty": "advanced", "score": 40}
    ],
    "vr": [
        {"name": "3D Basics", "difficulty": "beginner", "score": 15},
        {"name": "Virtual Reality Concepts", "difficulty": "beginner", "score": 15},
        {"name": "Basic Game Design", "difficulty": "intermediate", "score": 25},
        {"name": "Unity Basics", "difficulty": "intermediate", "score": 25},
        {"name": "3D Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Basic VR Interactions", "difficulty": "advanced", "score": 40},
        {"name": "Advanced VR Development", "difficulty": "advanced", "score": 40},
        {"name": "Unreal Engine", "difficulty": "advanced", "score": 40},
        {"name": "VR Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "game development": [
        {"name": "Game Design Basics", "difficulty": "beginner", "score": 15},
        {"name": "Unity Basics", "difficulty": "beginner", "score": 15},
        {"name": "Unreal Engine Basics", "difficulty": "intermediate", "score": 25},
        {"name": "2D Game Development", "difficulty": "intermediate", "score": 25},
        {"name": "3D Game Development", "difficulty": "advanced", "score": 40},
        {"name": "Game Physics", "difficulty": "advanced", "score": 40},
        {"name": "AI in Games", "difficulty": "advanced", "score": 40},
        {"name": "Multiplayer Game Development", "difficulty": "advanced", "score": 40},
        {"name": "VR Game Development", "difficulty": "advanced", "score": 40}
    ]
}

# Data structures to store members and leaders
members = defaultdict(list)
leaders = defaultdict(list)
user_data = {}
registered_users = set()
user_track_registrations = {}
matched_members = set()  # New set to track matched members
leader_departments = {}  # New dictionary to store leaders' departments

# Data storage functions
def save_data():
    try:
        # Convert member tuples to serializable format
        serialized_members = {}
        for track, member_list in members.items():
            serialized_members[track] = []
            for rating, reg_time, member in member_list:
                # Store user ID and name instead of User object
                serialized_member = {
                    'user_id': member.get('user_id') or member['user'].id,  # Handle both new and old format
                    'user_name': member.get('user_name') or member['user'].name,  # Handle both new and old format
                    'track': member['track'],
                    'rating': member['rating'],
                    'comment': member['comment'],
                    'department': member['department'],
                    'university': member['university'],
                    'selected_topics': member['selected_topics'],
                    'registration_time': member['registration_time']
                }
                serialized_members[track].append((rating, reg_time, serialized_member))

        # Convert leaders to serializable format
        serialized_leaders = {}
        for track, leader_list in leaders.items():
            serialized_leaders[track] = []
            for leader in leader_list:
                # Store user ID and name instead of User object
                serialized_leader = {
                    'user_id': leader.get('user_id') or leader['user'].id,  # Handle both new and old format
                    'user_name': leader.get('user_name') or leader['user'].name,  # Handle both new and old format
                    'team_name': leader['team_name'],
                    'track': leader['track'],
                    'team_comment': leader['team_comment'],
                    'department': leader['department'],
                    'university': leader['university']
                }
                serialized_leaders[track].append(serialized_leader)

        data = {
            'members': serialized_members,
            'leaders': serialized_leaders,
            'user_data': user_data,
            'registered_users': list(registered_users),
            'matched_members': list(matched_members),
            'leader_departments': leader_departments
        }
        
        with open('bot_data.json', 'w') as f:
            json.dump(data, f, default=str)
        logger.info("Data saved successfully")

    except Exception as e:
        logger.error(f"Error saving data: {str(e)}")

def load_data():
    try:
        if os.path.exists('bot_data.json'):
            with open('bot_data.json', 'r') as f:
                data = json.load(f)
            
            # Restore data structures
            global members, leaders, user_data, registered_users, matched_members, leader_departments
            
            # Restore members
            members = defaultdict(list)
            for track, member_list in data['members'].items():
                for rating, reg_time, member_dict in member_list:
                    # Store just the raw data - we'll get the User object when needed
                    member = {
                        'user_id': member_dict['user_id'],
                        'user_name': member_dict['user_name'],
                        'track': member_dict['track'],
                        'rating': member_dict['rating'],
                        'comment': member_dict['comment'],
                        'department': member_dict['department'],
                        'university': member_dict['university'],
                        'selected_topics': member_dict['selected_topics'],
                        'registration_time': member_dict['registration_time']
                    }
                    members[track].append((rating, reg_time, member))
            
            # Restore leaders
            leaders = defaultdict(list)
            for track, leader_list in data['leaders'].items():
                for leader_dict in leader_list:
                    # Store just the raw data
                    leader = {
                        'user_id': leader_dict['user_id'],
                        'user_name': leader_dict['user_name'],
                        'team_name': leader_dict['team_name'],
                        'track': leader_dict['track'],
                        'team_comment': leader_dict['team_comment'],
                        'department': leader_dict['department'],
                        'university': leader_dict['university']
                    }
                    leaders[track].append(leader)
            
            user_data = data['user_data']
            registered_users = set(data['registered_users'])
            matched_members = set(data['matched_members'])
            leader_departments = data['leader_departments']
            
            logger.info("Data loaded successfully")
    except Exception as e:
        logger.error(f"Error loading data: {str(e)}")

# Automatic restart function
def schedule_restart():
    save_data()
    logger.info("Initiating scheduled restart")
    
    python_executable = sys.executable  # Get Python executable path
    script_path = os.path.abspath(sys.argv[0])  # Get absolute path of script

    # Ensure proper quoting of file paths (handles spaces correctly)
    command = [python_executable, script_path]
    os.execv(python_executable, command)

# Error handling decorator
def error_handler(func):
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except Exception as e:
            ctx = args[0] if args else None
            error_msg = f"Error in {func.__name__}: {str(e)}"
            logger.error(error_msg)
            if ctx:
                await ctx.send(f"An error occurred. Please try again or contact support.\nError: {str(e)}")
    return wrapper

# Use these in your command responses, for example:
@bot.command(name="helpbot")
async def helpbot(ctx):
    help_title = "Discord Team Matching Bot Commands"
    help_description = (
        "Welcome to the Team Matching Bot! Here are the available commands:"
    )
    
    fields = [
        ("🔹 Registration Process", "`!start` - Complete entire registration process", False),
        ("🔹 Commands", 
         "`!choose_department` - Select department\n"
         "`!choose_role` - Choose Leader/Member\n"
         "`!choose_track` - Select track\n"
         "`!choose_topics` - Select topics\n"
         "`!write_comment` - Add comment", False),
        ("📌 Registration Flow",
         "1. Choose Department\n"
         "2. Choose Role\n"
         "3. Select Track Category\n"
         "4. Select Specific Track\n"
         "5. Choose Topics\n"
         "6. Write Comment", False)
    ]
    
    embed = format_embed_message(help_title, help_description, fields)
    await ctx.send(embed=embed)

# Helper function to calculate rating based on difficulty scores
def calculate_rating(user_id):
    track = user_data[user_id]['track']
    selected_topics = user_data[user_id].get('selected_topics', [])
    
    total_score = 0
    # Calculate total points from selected topics
    for topic_name in selected_topics:
        for topic in track_topics[track]:
            if topic['name'] == topic_name:
                total_score += topic['score']
    
    # Calculate total possible points across all topics
    total_possible_score = sum(topic['score'] for topic in track_topics[track])
    
    # Calculate rating as percentage
    return 0 if total_possible_score == 0 else min(int((total_score / total_possible_score) * 100), 100)

# Update the departments in choose_department command
@bot.command(name="choose_department")
async def choose_department(ctx):
    user_id = ctx.author.id
    role = user_data.get(user_id, {}).get('role')
    university = user_data.get(user_id, {}).get('university')

    # Check if the user is a leader and already has registered before
    if role == "leader" and user_id in leader_departments:
        stored_dept = leader_departments[user_id]['department']
        stored_univ = leader_departments[user_id]['university']
        
        # If university doesn't match previous registration
        if university != stored_univ:
            await ctx.send(f"You must register with your original university: {stored_univ}")
            user_data[user_id]["university"] = stored_univ
            await ctx.invoke(bot.get_command("choose_department"))
            return
            
        # Use the stored department and prevent changes
        user_data[user_id]["department"] = stored_dept
        await ctx.send(f"Your department is already set to {stored_dept.upper()} and cannot be changed.")
        await ctx.invoke(bot.get_command("choose_role"))
        return

    # If the leader doesn't have a department yet or is a member, allow them to choose
    select = Select(
        placeholder="Choose your department",
        options=[
            discord.SelectOption(label="CS", value="cs"),
            discord.SelectOption(label="IT", value="it"),
            discord.SelectOption(label="IS", value="is"),
            discord.SelectOption(label="AI", value="ai"),
            discord.SelectOption(label="SW", value="sw"),
            discord.SelectOption(label="BIO", value="bio")
        ]
    )

    async def callback(interaction):
        user_data[user_id]["department"] = select.values[0]
        
        # If the user is a leader, store their department and university permanently
        if role == "leader":
            leader_departments[user_id] = {
                'department': select.values[0],
                'university': user_data[user_id].get('university')
            }
        
        await interaction.response.send_message(f"You have chosen {select.values[0].upper()} department.")
        await ctx.invoke(bot.get_command("choose_role"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Please choose your department:", view=view)


# Add new command for university selection
@bot.command(name="choose_university")
async def choose_university(ctx):
    user_id = ctx.author.id
    role = user_data.get(user_id, {}).get('role')

    # Check if leader already registered
    if role == "leader" and user_id in leader_departments:
        stored_univ = leader_departments[user_id]['university']
        user_data[user_id]["university"] = stored_univ
        await ctx.send(f"Your university is already set to {stored_univ} and cannot be changed.")
        await ctx.invoke(bot.get_command("choose_department"))
        return

    # Create select menu with universities
    select = Select(
        placeholder="Choose your university",
        options=[discord.SelectOption(label=univ, value=univ) for univ in universities]
    )

    async def callback(interaction):
        user_data[user_id] = {"university": select.values[0]}
        await interaction.response.send_message(f"You have chosen {select.values[0]}.")
        await ctx.invoke(bot.get_command("choose_department"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Please choose your university:", view=view)

# Role Selection Command
@bot.command(name="choose_role")
async def choose_role(ctx):
    user_id = ctx.author.id
    
    select = Select(
        placeholder="Choose your role",
        options=[
            discord.SelectOption(label="Leader", value="leader"),
            discord.SelectOption(label="Member", value="member")
        ]
    )

    async def callback(interaction):
        selected_role = select.values[0]
        
        # Check if user is choosing to be a leader and has registered before
        if selected_role == "leader" and user_id in leader_departments:
            stored_univ = leader_departments[user_id]['university']
            stored_dept = leader_departments[user_id]['department']
            current_univ = user_data[user_id].get('university')
            current_dept = user_data[user_id].get('department')
            
            # Verify university and department match previous registration
            if current_univ != stored_univ or current_dept != stored_dept:
                await interaction.response.send_message(
                    f"⚠️ You must register with your original university ({stored_univ}) "
                    f"and department ({stored_dept.upper()}). Please start over with !start"
                )
                return
        
        user_data[user_id]["role"] = selected_role
        
        if selected_role == "leader":
            await interaction.response.send_message("You have chosen to be a Leader.")
            await ctx.send("Please enter your team name:")
            
            def check(m):
                return m.author == ctx.author and m.channel == ctx.channel
            
            try:
                msg = await bot.wait_for("message", check=check, timeout=60)
                team_name = msg.content
                user_data[user_id]["team_name"] = team_name
                await ctx.send(f"Your team name is {team_name}.")
                
                # Store leader's information if this is their first registration
                if user_id not in leader_departments:
                    leader_departments[user_id] = {
                        'university': user_data[user_id]['university'],
                        'department': user_data[user_id]['department']
                    }
                
                await ctx.invoke(bot.get_command("choose_track"))
            except asyncio.TimeoutError:
                await ctx.send("Team name selection timed out. Please try again.")
        
        else:  # Member
            await interaction.response.send_message("You have chosen to be a Member.")
            await ctx.invoke(bot.get_command("choose_track"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Choose your role:", view=view)

# Track Selection Command
@bot.command(name="choose_track")
async def choose_track(ctx):
    role = user_data.get(ctx.author.id, {}).get('role')
    
    select = Select(
        placeholder="Choose track category",
        options=[discord.SelectOption(label=category, value=category) for category in track_categories.keys()]
    )

    async def category_callback(interaction):
        category = select.values[0]
        track_options = [
            discord.SelectOption(label=track, value=track) 
            for track in track_categories[category]
        ]
        
        track_select = Select(
            placeholder=f"Choose track in {category}",
            options=track_options
        )

        async def track_callback(track_interaction):
            selected_track = track_select.values[0]
            
            # Remove previous track restrictions for leaders
            user_data[ctx.author.id]['track'] = selected_track
            await track_interaction.response.send_message(f"You have chosen {selected_track}.")
            
            # Different flow for leader and member
            if role == "leader":
                await ctx.invoke(bot.get_command("write_comment"))
            else:
                await ctx.invoke(bot.get_command("choose_topics"))

        track_select.callback = track_callback
        track_view = View()
        track_view.add_item(track_select)
        await interaction.response.send_message(f"Choose a track in {category}:", view=track_view)

    select.callback = category_callback
    view = View()
    view.add_item(select)
    await ctx.send("Choose a track category:", view=view)

# Topic Selection Command
@bot.command(name="choose_topics")
async def choose_topics(ctx):
    track = user_data[ctx.author.id]["track"]
    topics = [topic['name'] for topic in track_topics[track]]
    
    select = Select(
        placeholder="Choose topics you've studied",
        options=[discord.SelectOption(label=topic, value=topic) for topic in topics],
        max_values=len(topics)
    )

    async def callback(interaction):
        selected_topics = select.values
        user_data[ctx.author.id]["selected_topics"] = selected_topics
        rating = calculate_rating(ctx.author.id)
        user_data[ctx.author.id]["rating"] = rating
        
        await interaction.response.send_message(f"You have selected: {', '.join(selected_topics)}. Your rating is {rating}%.")
        await ctx.invoke(bot.get_command("write_comment"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Please select topics you've studied:", view=view)

# Write Comment Command
@bot.command(name="write_comment")
async def write_comment(ctx):
    role = user_data[ctx.author.id]["role"]
    
    if role == "member" and ctx.author.id in registered_users:
        await ctx.send("You have already registered. Wait for matching.")
        return

    comment_prompt = (
        "Write A Comment About Your Team Like Members In It And Your Project Idea.." 
        if role == "leader" else 
        "Write A Comment About Yourself Like What You Have Studied And What You Will Study In The Future.."
    )
    
    await ctx.send(comment_prompt)

    def check(m):
        return m.author == ctx.author and m.channel == ctx.channel

    try:
        msg = await bot.wait_for("message", check=check, timeout=60)
        comment = msg.content
        user_data[ctx.author.id]["comment"] = comment
        registration_time = time.time()
        
        track = user_data[ctx.author.id]["track"]
        department = user_data[ctx.author.id]["department"]
        university = user_data[ctx.author.id]["university"]
        
        if role == "member":
            member = {
                "user_id": ctx.author.id,  # Store ID instead of User object
                "user_name": ctx.author.name,  # Store name separately
                "track": track,
                "rating": user_data[ctx.author.id]["rating"],
                "comment": comment,
                "department": department,
                "university": university,
                "selected_topics": user_data[ctx.author.id].get("selected_topics", []),
                "registration_time": registration_time
            }
            heapq.heappush(members[track], (-member["rating"], registration_time, member))
            registered_users.add(ctx.author.id)
        else:
            leader = {
                "user_id": ctx.author.id,  # Store ID instead of User object
                "user_name": ctx.author.name,  # Store name separately
                "team_name": user_data[ctx.author.id]["team_name"],
                "track": track,
                "team_comment": comment,
                "department": department,
                "university": university
            }
            leaders[track].append(leader)
        
        await ctx.send("Your registration is complete. You've been added to the matching queue.")
        await ctx.invoke(bot.get_command("match"))
        
    except asyncio.TimeoutError:
        await ctx.send("Comment submission timed out. Please try again.")

# Update the start command to begin with university selection
# Add error handling to existing commands
@bot.command(name="start")
@error_handler
@commands.cooldown(1, 15, BucketType.user)
async def start(ctx):
    user_id = ctx.author.id
    
    # If user is already registered as a leader, verify their status
    if user_id in leader_departments:
        stored_data = leader_departments[user_id]
        await ctx.send(
            f"⚠️ You have previously registered as a leader for:\n"
            f"University: {stored_data['university']}\n"
            f"Department: {stored_data['department'].upper()}\n\n"
            f"You must use these same details to register again."
        )
    
    # Check if the member is already registered and not matched
    if user_id in registered_users:
        await ctx.send("You have already registered. Wait for matching.")
        return

    # Clear any existing data for the user
    if user_id in user_data:
        del user_data[user_id]
    
    await ctx.invoke(bot.get_command("choose_university"))
    save_data()

#Perform_Matching Process Function
async def perform_matching():
    try:
        matching_track_count = 0
        
        for category, tracks in track_categories.items():
            for track in tracks:
                track_leaders = leaders.get(track, []).copy()
                track_members = members.get(track, [])
                
                while track_leaders and track_members:
                    try:
                        leader = track_leaders[0]  # Don't pop yet, wait until match is confirmed
                        matching_member = None
                        temp_members = []
                        
                        while track_members:
                            rating, reg_time, member = heapq.heappop(track_members)
                            
                            # Check if member is still in registered_users
                            if (member['user_id'] in registered_users and 
                                leader["university"] == member["university"] and 
                                leader["department"] == member["department"]):
                                matching_member = member
                                break
                            else:
                                temp_members.append((rating, reg_time, member))
                        
                        # Restore unmatched members back to the heap
                        for m in temp_members:
                            heapq.heappush(track_members, m)
                        
                        if not matching_member:
                            track_leaders.pop(0)  # Remove leader if no match found
                            continue

                        # Get the actual Discord User objects
                        try:
                            leader_user = await bot.fetch_user(leader['user_id'])
                            member_user = await bot.fetch_user(matching_member['user_id'])
                            
                            if not leader_user or not member_user:
                                logger.error("Could not fetch user objects")
                                continue
                                
                            leader_channel = await leader_user.create_dm()
                            member_channel = await member_user.create_dm()
                        except discord.HTTPException as e:
                            logger.error(f"Failed to create DM channels: {e}")
                            continue

                        # Format messages
                        success_header = "```ansi\n\u001b[1;32m🎉 MATCHING SUCCESS! 🎉\u001b[0m\n```"
                        
                        leader_match_info = (
                            f"{success_header}\n"
                            f"**📋 Match Details**\n"
                            f"```yml\n"
                            f"Member Name   : {member_user.name}\n"
                            f"University    : {matching_member['university']}\n"
                            f"Department    : {matching_member['department'].upper()}\n"
                            f"Track         : {matching_member['track']}\n"
                            f"Team          : {leader['team_name']}\n"
                            f"```\n"
                        )

                        member_match_info = (
                            f"{success_header}\n"
                            f"**📋 Match Details**\n"
                            f"```yml\n"
                            f"Team Leader   : {leader_user.name}\n"
                            f"University    : {leader['university']}\n"
                            f"Department    : {leader['department'].upper()}\n"
                            f"Track         : {leader['track']}\n"
                            f"Team          : {leader['team_name']}\n"
                            f"```\n"
                        )

                        team_info = (
                            f"**🏢 Team Information**\n"
                            f"```ansi\n"
                            f"\u001b[1;34m── Team Leader's Message ──\u001b[0m\n"
                            f"{leader['team_comment']}\n"
                            f"```"
                        )

                        topics_str = ", ".join(matching_member["selected_topics"])
                        member_profile = (
                            f"**👤 Member Profile**\n"
                            f"```ansi\n"
                            f"\u001b[1;33m── Technical Background ──\u001b[0m\n"
                            f"• Track: {matching_member['track']}\n"
                            f"• Rating: {matching_member['rating']}%\n\n"
                            f"\u001b[1;33m── Topics Studied ──\u001b[0m\n"
                            f"{topics_str}\n\n"
                            f"\u001b[1;33m── Personal Note ──\u001b[0m\n"
                            f"{matching_member['comment']}\n"
                            f"```"
                        )

                        contact_info = (
                            f"**📱 Next Steps**\n"
                            f"```ansi\n"
                            f"\u001b[1;35mYou can now communicate directly through Discord!\u001b[0m\n"
                            f"Feel free to discuss project details and next steps.\n"
                            f"```"
                        )

                        # Send all messages with error handling
                        try:
                            await leader_channel.send(leader_match_info)
                            await leader_channel.send(team_info)
                            await leader_channel.send(member_profile)
                            await leader_channel.send(contact_info)
                            
                            await member_channel.send(member_match_info)
                            await member_channel.send(team_info)
                            await member_channel.send(member_profile)
                            await member_channel.send(contact_info)
                            
                            # Only proceed with cleanup if messages were sent successfully
                            # Clean up after successful match
                            leader_id = leader['user_id']
                            member_id = matching_member['user_id']
                            
                            track_leaders.pop(0)  # Now safe to remove the leader
                            leaders[track] = [l for l in leaders[track] if l['user_id'] != leader_id]
                            
                            if leader_id in user_data:
                                del user_data[leader_id]
                            if member_id in user_data:
                                del user_data[member_id]
                            
                            registered_users.discard(member_id)
                            matched_members.add(member_id)
                            
                            if leader_id in leader_departments:
                                del leader_departments[leader_id]
                            
                            matching_track_count += 1
                            logger.info(f"Successful match in track {track}: Leader {leader_id} with Member {member_id}")
                            
                        except discord.HTTPException as e:
                            logger.error(f"Failed to send match messages: {e}")
                            continue
                        except Exception as e:
                            logger.error(f"Error during match cleanup: {e}")
                            continue
                        
                    except Exception as e:
                        logger.error(f"Error in matching process for track {track}: {str(e)}")
                        continue
        
        # Save data after all matches are complete
        try:
            save_data()
        except Exception as e:
            logger.error(f"Error saving data after matching: {str(e)}")
        
        return matching_track_count
        
    except Exception as e:
        logger.error(f"Error in perform_matching: {str(e)}")
        return 0


# Add these helper functions for improved message formatting
def format_section_header(title, emoji=""):
    return f"```ansi\n\u001b[1;36m{emoji} {title} {emoji}\u001b[0m\n```"

def format_field(label, value):
    return f"{label:<12}: {value}"

# Message formatting functions
def format_embed_message(title, description, fields=None, color=0x3498db):
    embed = discord.Embed(
        title=title,
        description=description,
        color=color
    )
    
    if fields:
        for name, value, inline in fields:
            embed.add_field(name=name, value=value, inline=inline)
    
    return embed

def format_success_message(message):
    return f"```ansi\n\u001b[1;32m✅ {message}\u001b[0m\n```"

def format_error_message(message):
    return f"```ansi\n\u001b[1;31m❌ {message}\u001b[0m\n```"

def format_info_message(message):
    return f"```ansi\n\u001b[1;34mℹ️ {message}\u001b[0m\n```"

def format_warning_message(message):
    return f"```ansi\n\u001b[1;33m⚠️ {message}\u001b[0m\n```"

# Match Command with Enhanced Matching and Communication
@bot.command(name="match")
async def match(ctx):
    bot.loop.create_task(perform_matching())
    await ctx.send("Matching process started in the background.")


# Automatic Matching Task
@tasks.loop(seconds=30)
async def auto_match():
    await perform_matching()

# Start the auto-match task when the bot is ready
# Update the bot's event handlers
@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')
    logger.info(f'Bot logged in as {bot.user}')
    load_data()  # Load saved data
    auto_match.start()
    
    # Schedule restart every hour
    schedule.every(45).minutes.do(schedule_restart)
    
    # Start schedule checker
    while True:
        schedule.run_pending()
        await asyncio.sleep(30)

@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandNotFound):
        await ctx.send("```❌ Invalid command. Use `!helpbot` for a list of commands.```")
    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send("```❌ Missing required argument. Please check the command usage.```")
    elif isinstance(error, commands.CommandOnCooldown):
        await ctx.send(f"```⏳ Please wait {error.retry_after:.2f}s before using this command again.```")
    else:
        error_msg = f"Command error: {str(error)}"
        logger.error(error_msg)
        await ctx.send(f"```❌ An error occurred. Please try again later.\nError: {str(error)}```")
    

# Run the bot
# write this if you use terminal or cmd : set DISCORD_BOT_TOKEN=your_bot_token_here
# write this if you use powershell : $env:DISCORD_BOT_TOKEN="your_bot_token_here"
# Update the main function
async def main():
    try:
        # Handle shutdown gracefully
        def signal_handler(sig, frame):
            logger.info("Shutdown signal received")
            save_data()
            sys.exit(0)
        
        signal.signal(signal.SIGINT, signal_handler)
        signal.signal(signal.SIGTERM, signal_handler)
        
        async with bot:
            await bot.start(os.getenv("DISCORD_BOT_TOKEN"))
    except Exception as e:
        logger.error(f"Main function error: {str(e)}")
        sys.exit(1)

if __name__ == "__main__":
    asyncio.run(main())
```

#### 1. **Imports 📦**

The first thing we see is a set of imports. These are essential libraries for the bot’s operation:

```python
import json
import logging
from datetime import datetime
import schedule
import signal
from logging.handlers import RotatingFileHandler
import discord
from discord.ext import commands, tasks
from discord.ext.commands import cooldown, BucketType
from discord.ui import Button, View, Select
from collections import defaultdict
import heapq
import asyncio
import os
import sys
from dotenv import load_dotenv
import time
```

- **`JOSN`**: Used to read and write JSON data, which the bot uses for saving user data and matching results.
- **`logging`**: Handles the logging of the bot’s activity, helpful for debugging and tracking bot behavior.
- **`datetime`**: Used for working with dates and times. It helps in logging timestamps and tracking when certain events (like user registration or team matching) occur.
- **`schedule`**: This library is used for scheduling tasks, such as automatically restarting the bot at a specified interval. In the bot’s code, it helps schedule the restart process.
- **`signal`**: This is used to handle system signals. It is typically used for graceful shutdowns and handling termination signals in the bot's operation.
- **`RotatingFileHandler`** (from `logging.handlers`): This handler is used to write log entries into a log file. The file is rotated when it reaches a certain size (`5MB` in this case), ensuring that the log doesn’t become too large and unmanageable.
- **`discord`**: The core library for interacting with Discord’s API. It allows the bot to send and receive messages, manage users, create channels, and interact with other Discord features.
- **`discord.ext.commands`**: This extension module of `discord.py` is used to manage bot commands. It simplifies the process of creating, handling, and executing commands (like `!start`, `!choose_role`, etc.).
- **`discord.ext.tasks`**: This extension module helps run background tasks, like performing automated actions at regular intervals. The bot uses this to periodically check for and perform team matching.
- **`discord.ui`**: Contains useful utilities for creating Discord UI components like buttons, selects, and views, which are used in the bot for interactions such as selecting a department, choosing roles, and matching teams.
- **`defaultdict`** (from `collections`): This is a dictionary that provides a default value for nonexistent keys. It's useful in the bot for managing data structures such as user registration, team assignments, and other dynamic values.
- **`heapq`**: A library used for creating heaps (binary trees). In the bot, it’s used for maintaining a priority queue, which helps with efficiently matching users based on their registration times and ratings.
- **`asyncio`**: A library for asynchronous programming. It allows the bot to handle multiple operations concurrently, such as waiting for user input while still running other tasks in the background (e.g., performing team matching).
- **`os`**: Provides a way to interact with the operating system, like file and directory management, environment variable handling, and process control.
- **`sys`**: Used to interact with the system's environment and exit the program gracefully when necessary, like when the bot is shutting down.
- **`load_dotenv`** (from `dotenv`): This function loads environment variables from a `.env` file, which is where sensitive information (like the Discord bot token) is stored. It ensures that this information is securely accessed at runtime.
- **`time`**: Used for handling time-related tasks, including tracking the duration of user actions, such as registration time.

These libraries work together to provide a complete, efficient, and well-structured bot that can handle various tasks like logging, scheduling, interacting with users, and performing the team matching process.


#### 2. **Logging Setup 📜**

The logging setup is crucial for tracking the bot’s activity and errors. The bot logs important events, such as user registrations, matches, errors, and system activities. Here's how the logging is configured:

```python
log_formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
log_file = 'bot_logs.log'
log_handler = RotatingFileHandler(log_file, maxBytes=5*1024*1024, backupCount=5)
log_handler.setFormatter(log_formatter)

logger = logging.getLogger('BotLogger')
logger.setLevel(logging.INFO)
logger.addHandler(log_handler)
```

The logging setup begins by defining a custom format for how the bot's logs will appear. The `log_formatter` is created using the `logging.Formatter` class. This formatter specifies that each log message will include the following information:

- **Timestamp (`%(asctime)s`)**: This adds the exact time when the log entry was made. It’s crucial for debugging because it shows when certain actions or errors occurred.
- **Log Level (`%(levelname)s`)**: The log level indicates the severity of the log message. This could be one of several levels such as `INFO`, `WARNING`, `ERROR`, or `DEBUG`. Each level helps categorize the importance of the message.
- **Message (`%(message)s`)**: The actual log message that is being recorded, which could be an error, event, or other important information.

Next, a `RotatingFileHandler` is set up for logging. This handler will log messages to a file named `bot_logs.log`. The handler has two important properties:

- **Log File (`log_file = 'bot_logs.log'`)**: The log messages will be stored in this file, allowing you to refer to them later.
- **File Rotation**: The `RotatingFileHandler` will ensure that the log file doesn't get too large. When the file size reaches 5MB, it will automatically create a new file, keeping the last 5 backups. This ensures that old logs are kept and the log file doesn't consume too much space.

The handler is then added to the logger using `logger.addHandler(log_handler)`. This ties the handler to the logger so that the logs will be written to the file specified in the handler (`bot_logs.log`). 

This logging setup is essential for monitoring the bot’s performance, catching errors, and debugging issues.


#### 3. **Initializing the Bot 🤖**

The next step in the bot’s code is to load environment variables, which is crucial for managing sensitive information like the bot token. This is done using the `load_dotenv()` function. 

```python
# Load environment variables from .env file
load_dotenv()
```

- **`load_dotenv()`**: This function loads environment variables from a `.env` file into the Python environment. The `.env` file contains sensitive information like the Discord bot’s token (`DISCORD_BOT_TOKEN`). By using this method, we ensure that the token is not exposed directly in the source code, providing better security.

The next section involves setting up the bot’s **intents**. Intents are permissions that the bot needs to access certain information from Discord. For example, to get messages, member data, etc.

```python
# Initialize the bot
intents = discord.Intents.default()
intents.message_content = True
intents.members = True
bot = commands.Bot(command_prefix="!", intents=intents , heartbeat_timeout=120)
```

Here's the breakdown:

- **`intents = discord.Intents.default()`**: This initializes the default intents for the bot. It allows the bot to access basic events like member updates or message content.
- **`intents.message_content = True`**: This explicitly enables the bot to read the content of messages. Without this, the bot wouldn't be able to see messages unless specifically authorized in the Discord Developer Portal.
- **`intents.members = True`**: This grants the bot permission to read member data such as joining, leaving, and updating member roles.

After defining the necessary intents, the bot is initialized using the `commands.Bot` class:

- **`bot = commands.Bot(command_prefix="!", intents=intents , heartbeat_timeout=120)`**: This creates a new bot instance with the specified **command prefix** (`!`). This means that all bot commands will begin with `!` (e.g., `!start`, `!help`). 
  - **`intents=intents`**: Passes the intents defined earlier, ensuring the bot can access the necessary data.
  - **`heartbeat_timeout=120`**: This defines the timeout for heartbeat messages sent to Discord. If the bot fails to send a heartbeat within this time frame, it will disconnect.

The `bot` instance will now handle all incoming commands and interactions from users in Discord, with the defined intents and settings.


#### 4. **Track Categories, Universities, and Topics 📚**

```python
# Categorized Tracks
track_categories = {
    "Backend Frameworks": [".net", "node.js", "laravel", "django", "spring"],
    "Cybersecurity Specializations": ["network security", "ethical hacking", "digital forensics"],
    "Data Science Specializations": ["machine learning", "data analysis", "data engineering", "deep learning"],
    "Other Tracks": ["front end", "ui-ux", "flutter", "cloud", "mobile", "embedded systems", "vr", "game development"]
}

universities = [
    "Cairo University",
    "Ain Shams University",
    "Alexandria University",
    "Helwan University",
    "Mansoura University",
    "Assiut University",
    "Zagazig University",
    "Tanta University",
    "Suez Canal University",
    "Benha University",
    "Fayoum University",
    "South Valley University",
    "Menoufia University",
    "Port Said University",
    "Beni Suef University",
    "Kafrelsheikh University",
    "Damietta University",
    "Sohag University",
    "Modern Academy",
    "MSA University",
    "MTI University",
    "Future University",
    "October 6 University",
    "Badr University",
    "New Cairo Academy"
]
```

In this section of the code, the bot is set up with a predefined list of **track categories** and the corresponding **universities**. These are important for the registration process, where users will select their department, track, and university.

- **Track Categories (`track_categories`)**: This is a dictionary that categorizes the various fields or tracks that users can choose from, such as Backend Frameworks, Cybersecurity, and Data Science. Each category contains a list of specific tracks (e.g., `.net`, `node.js`, `machine learning`).
  - The track categories are organized into:
    - **Backend Frameworks**: Contains tracks like `.net`, `node.js`, `django`, etc.
    - **Cybersecurity Specializations**: Includes tracks like `network security`, `ethical hacking`, etc.
    - **Data Science Specializations**: For tracks like `machine learning`, `data analysis`, etc.
    - **Other Tracks**: A catch-all category for various tracks such as `flutter`, `front end`, `game development`, etc.

This categorization is useful when users are asked to select their track during registration. It helps guide them through the process and ensures they can only choose from relevant tracks based on their area of study.

- **Universities (`universities`)**: This list contains the names of various universities that users can select from. The list includes both real-world universities like Cairo University, Ain Shams University, and Alexandria University, as well as others that may be applicable to a specific region or institution.
  
  By offering a set of predefined universities, the bot ensures that all users are grouped according to their university, which can help when matching users for team formation. Users must select their correct university, which ensures that teams are formed from members within the same institution.

#### 5. **Track Topics with Difficulty Scores 📝**

This section defines the topics under each track, along with a **difficulty score** and **rating** for each topic. 
```python
# Track Topics with Difficulty Scores
track_topics = {
    ".net": [
        {"name": "C# Basics", "difficulty": "beginner", "score": 15},
        {"name": ".NET Core Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Web API", "difficulty": "intermediate", "score": 25},
        {"name": ".NET MVC", "difficulty": "intermediate", "score": 25},
        {"name": "Entity Framework", "difficulty": "advanced", "score": 40},
        {"name": "Dependency Injection", "difficulty": "advanced", "score": 40},
        {"name": "Microservices with .NET", "difficulty": "advanced", "score": 40},
        {"name": "Advanced ORM", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "node.js": [
        {"name": "JavaScript Basics", "difficulty": "beginner", "score": 15},
        {"name": "Node.js Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Express.js Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Async Programming", "difficulty": "intermediate", "score": 25},
        {"name": "RESTful API Design", "difficulty": "advanced", "score": 40},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Express", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Tuning", "difficulty": "advanced", "score": 40}
    ],
    "laravel": [
        {"name": "PHP Basics", "difficulty": "beginner", "score": 15},
        {"name": "Laravel Installation", "difficulty": "beginner", "score": 15},
        {"name": "Routing Fundamentals", "difficulty": "intermediate", "score": 25},
        {"name": "Eloquent ORM", "difficulty": "intermediate", "score": 25},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "Blade Templates", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Eloquent", "difficulty": "advanced", "score": 40},
        {"name": "Laravel Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "django": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Django Setup", "difficulty": "beginner", "score": 15},
        {"name": "Basic Models", "difficulty": "intermediate", "score": 25},
        {"name": "Django ORM", "difficulty": "intermediate", "score": 25},
        {"name": "Authentication", "difficulty": "advanced", "score": 40},
        {"name": "REST Framework", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Querying", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "spring": [
        {"name": "Java Basics", "difficulty": "beginner", "score": 15},
        {"name": "Spring Boot Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Dependency Injection", "difficulty": "intermediate", "score": 25},
        {"name": "Spring MVC", "difficulty": "intermediate", "score": 25},
        {"name": "JPA", "difficulty": "advanced", "score": 40},
        {"name": "Security Configuration", "difficulty": "advanced", "score": 40},
        {"name": "Microservices", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Caching", "difficulty": "advanced", "score": 40},
        {"name": "Performance Tuning", "difficulty": "advanced", "score": 40}
    ],
    "network security": [
        {"name": "Network Basics", "difficulty": "beginner", "score": 15},
        {"name": "TCP/IP", "difficulty": "beginner", "score": 15},
        {"name": "Firewall Concepts", "difficulty": "intermediate", "score": 25},
        {"name": "Intrusion Detection", "difficulty": "intermediate", "score": 25},
        {"name": "Network Protocols", "difficulty": "advanced", "score": 40},
        {"name": "Packet Analysis", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Threat Detection", "difficulty": "advanced", "score": 40},
        {"name": "Network Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Secure Network Design", "difficulty": "advanced", "score": 40}
    ],
    "ethical hacking": [
        {"name": "Security Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Networking", "difficulty": "beginner", "score": 15},
        {"name": "Linux Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Penetration Testing Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Vulnerability Assessment", "difficulty": "advanced", "score": 40},
        {"name": "Exploit Techniques", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Penetration Testing", "difficulty": "advanced", "score": 40},
        {"name": "Red Team Tactics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Exploit Development", "difficulty": "advanced", "score": 40}
    ],
    "digital forensics": [
        {"name": "Computer Forensics Basics", "difficulty": "beginner", "score": 15},
        {"name": "Evidence Preservation", "difficulty": "beginner", "score": 15},
        {"name": "Basic Tools", "difficulty": "intermediate", "score": 25},
        {"name": "Forensic Analysis Techniques", "difficulty": "intermediate", "score": 25},
        {"name": "Disk Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Memory Forensics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Forensic Tools", "difficulty": "advanced", "score": 40},
        {"name": "Malware Analysis", "difficulty": "advanced", "score": 40},
        {"name": "Complex Investigation Techniques", "difficulty": "advanced", "score": 40}
    ],
    "machine learning": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Statistics Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic ML Algorithms", "difficulty": "intermediate", "score": 25},
        {"name": "Scikit-learn", "difficulty": "intermediate", "score": 25},
        {"name": "Supervised Learning", "difficulty": "advanced", "score": 40},
        {"name": "Feature Engineering", "difficulty": "advanced", "score": 40},
        {"name": "Deep Learning", "difficulty": "advanced", "score": 40},
        {"name": "Advanced ML Algorithms", "difficulty": "advanced", "score": 40},
        {"name": "Model Deployment", "difficulty": "advanced", "score": 40}
    ],
    "data analysis": [
        {"name": "Python Basics", "difficulty": "beginner", "score": 15},
        {"name": "Excel Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "Basic Statistics", "difficulty": "intermediate", "score": 25},
        {"name": "Pandas", "difficulty": "intermediate", "score": 25},
        {"name": "NumPy", "difficulty": "advanced", "score": 40},
        {"name": "Data Visualization", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Analytics", "difficulty": "advanced", "score": 40},
        {"name": "Predictive Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Big Data Tools", "difficulty": "advanced", "score": 40}
    ],
    "data engineering": [
        {"name": "SQL Basics", "difficulty": "beginner", "score": 15},
        {"name": "Data Warehousing Concepts", "difficulty": "beginner", "score": 15},
        {"name": "ETL Fundamentals", "difficulty": "intermediate", "score": 25},
        {"name": "Apache Spark", "difficulty": "intermediate", "score": 25},
        {"name": "Big Data Technologies", "difficulty": "advanced", "score": 40},
        {"name": "Data Pipeline Design", "difficulty": "advanced", "score": 40},
        {"name": "Distributed Computing", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Data Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Real-time Data Processing", "difficulty": "advanced", "score": 40}
    ],
    "deep learning": [
        {"name": "Neural Network Basics", "difficulty": "beginner", "score": 15},
        {"name": "Python for AI", "difficulty": "beginner", "score": 15},
        {"name": "Basic Deep Learning Concepts", "difficulty": "intermediate", "score": 25},
        {"name": "TensorFlow", "difficulty": "intermediate", "score": 25},
        {"name": "Keras", "difficulty": "advanced", "score": 40},
        {"name": "Neural Network Architectures", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Neural Networks", "difficulty": "advanced", "score": 40},
        {"name": "Computer Vision", "difficulty": "advanced", "score": 40},
        {"name": "NLP Techniques", "difficulty": "advanced", "score": 40}
    ],
    "front end": [
        {"name": "HTML Basics", "difficulty": "beginner", "score": 15},
        {"name": "CSS Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "JavaScript Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Responsive Design", "difficulty": "intermediate", "score": 25},
        {"name": "Bootstrap", "difficulty": "advanced", "score": 40},
        {"name": "JavaScript ES6+", "difficulty": "advanced", "score": 40},
        {"name": "React", "difficulty": "advanced", "score": 40},
        {"name": "Vue.js", "difficulty": "advanced", "score": 40},
        {"name": "State Management", "difficulty": "advanced", "score": 40}
    ],
    "ui-ux": [
        {"name": "Design Principles", "difficulty": "beginner", "score": 15},
        {"name": "Color Theory", "difficulty": "beginner", "score": 15},
        {"name": "Typography Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Wireframing", "difficulty": "intermediate", "score": 25},
        {"name": "Prototyping", "difficulty": "advanced", "score": 40},
        {"name": "User Research", "difficulty": "advanced", "score": 40},
        {"name": "Figma", "difficulty": "advanced", "score": 40},
        {"name": "Adobe XD", "difficulty": "advanced", "score": 40},
        {"name": "Advanced UX Strategy", "difficulty": "advanced", "score": 40}
    ],
    "flutter": [
        {"name": "Dart Basics", "difficulty": "beginner", "score": 15},
        {"name": "Flutter Installation", "difficulty": "beginner", "score": 15},
        {"name": "Basic Widgets", "difficulty": "intermediate", "score": 25},
        {"name": "State Management", "difficulty": "intermediate", "score": 25},
        {"name": "Navigation", "difficulty": "advanced", "score": 40},
        {"name": "API Integration", "difficulty": "advanced", "score": 40},
        {"name": "Custom Widgets", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40},
        {"name": "Advanced State Solutions", "difficulty": "advanced", "score": 40}
    ],
    "cloud": [
        {"name": "Cloud Computing Basics", "difficulty": "beginner", "score": 15},
        {"name": "Basic Networking", "difficulty": "beginner", "score": 15},
        {"name": "Virtual Machines", "difficulty": "intermediate", "score": 25},
        {"name": "AWS Basics", "difficulty": "intermediate", "score": 25},
        {"name": "Azure Fundamentals", "difficulty": "advanced", "score": 40},
        {"name": "Docker Basics", "difficulty": "advanced", "score": 40},
        {"name": "Kubernetes", "difficulty": "advanced", "score": 40},
        {"name": "Cloud Security", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Deployment Strategies", "difficulty": "advanced", "score": 40}
    ],
    "mobile": [
        {"name": "Mobile Development Basics", "difficulty": "beginner", "score": 15},
        {"name": "UI Design for Mobile", "difficulty": "beginner", "score": 15},
        {"name": "Android Development", "difficulty": "intermediate", "score": 25},
        {"name": "iOS Development", "difficulty": "intermediate", "score": 25},
        {"name": "React Native Basics", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Mobile Architectures", "difficulty": "advanced", "score": 40},
        {"name": "Performance Optimization", "difficulty": "advanced", "score": 40},
        {"name": "Cross-Platform Development", "difficulty": "advanced", "score": 40}
    ],
    "embedded systems": [
        {"name": "Electronics Basics", "difficulty": "beginner", "score": 15},
        {"name": "Microcontroller Fundamentals", "difficulty": "beginner", "score": 15},
        {"name": "C Programming", "difficulty": "intermediate", "score": 25},
        {"name": "Arduino Programming", "difficulty": "intermediate", "score": 25},
        {"name": "Raspberry Pi", "difficulty": "advanced", "score": 40},
        {"name": "Sensor Integration", "difficulty": "advanced", "score": 40},
        {"name": "IoT Protocols", "difficulty": "advanced", "score": 40},
        {"name": "Advanced Embedded Programming", "difficulty":"advanced", "score": 40},
        {"name": "Real-Time Systems", "difficulty": "advanced", "score": 40}
    ],
    "vr": [
        {"name": "3D Basics", "difficulty": "beginner", "score": 15},
        {"name": "Virtual Reality Concepts", "difficulty": "beginner", "score": 15},
        {"name": "Basic Game Design", "difficulty": "intermediate", "score": 25},
        {"name": "Unity Basics", "difficulty": "intermediate", "score": 25},
        {"name": "3D Modeling", "difficulty": "advanced", "score": 40},
        {"name": "Basic VR Interactions", "difficulty": "advanced", "score": 40},
        {"name": "Advanced VR Development", "difficulty": "advanced", "score": 40},
        {"name": "Unreal Engine", "difficulty": "advanced", "score": 40},
        {"name": "VR Performance Optimization", "difficulty": "advanced", "score": 40}
    ],
    "game development": [
        {"name": "Game Design Basics", "difficulty": "beginner", "score": 15},
        {"name": "Unity Basics", "difficulty": "beginner", "score": 15},
        {"name": "Unreal Engine Basics", "difficulty": "intermediate", "score": 25},
        {"name": "2D Game Development", "difficulty": "intermediate", "score": 25},
        {"name": "3D Game Development", "difficulty": "advanced", "score": 40},
        {"name": "Game Physics", "difficulty": "advanced", "score": 40},
        {"name": "AI in Games", "difficulty": "advanced", "score": 40},
        {"name": "Multiplayer Game Development", "difficulty": "advanced", "score": 40},
        {"name": "VR Game Development", "difficulty": "advanced", "score": 40}
    ]
}
```

- **Track Topics (`track_topics`)**: This is a dictionary where each key represents a track (e.g., `.net`, `node.js`, `machine learning`), and the value is a list of dictionaries that describe each topic under that track.
  - **`difficulty`**: Each topic has a difficulty level that could be `beginner`, `intermediate`, or `advanced`. This helps in evaluating a user's experience and skill level with the topic.
  - **`score`**: The `score` represents the weight or value of each topic. More advanced topics or topics with greater relevance may have a higher score. These scores are important when calculating the user’s overall rating, which will influence the team matching process.
  
For example:
- Under the `.net` track, topics like "C# Basics" and ".NET Core Fundamentals" are **beginner** topics with a score of 15, while topics like "Advanced ORM" and "Performance Optimization" are **advanced** topics with a score of 40.

These topics and scores play a crucial role in determining how well a user matches with others during the team formation phase.


#### 6. **Data Structures to Store Members and Leaders 💼**

In this section, the bot sets up data structures that are essential for storing and organizing the users’ data, including members, leaders, and other relevant information.
```python
# Data structures to store members and leaders
members = defaultdict(list)
leaders = defaultdict(list)
user_data = {}
registered_users = set()
user_track_registrations = {}
matched_members = set()  # New set to track matched members
leader_departments = {}  # New dictionary to store leaders' departments
```

- **`members = defaultdict(list)`**: This is a dictionary that stores lists of members for each track. A `defaultdict` is used here because it automatically initializes an empty list for any track that doesn’t already exist in the dictionary. 
  - **Key**: Track name (e.g., `".net"`, `"network security"`)
  - **Value**: A list of members who have selected that track. Each member is stored with relevant details such as their rating, registration time, and selected topics.

- **`leaders = defaultdict(list)`**: Similar to `members`, this dictionary stores the **leaders** for each track. Again, a `defaultdict` is used, ensuring that each track starts with an empty list of leaders.
  - **Key**: Track name (e.g., `"backend frameworks"`, `"machine learning"`)
  - **Value**: A list of leaders in that track, including their team names, comments, and other information relevant to the team formation process.

- **`user_data = {}`**: This dictionary stores individual user information, such as their role (leader or member), selected track, university, department, and other details that are required for registration and matching.
  - **Key**: User ID (the unique identifier for each Discord user)
  - **Value**: A dictionary containing the user’s details, including their role, selected topics, and track information.

- **`registered_users = set()`**: A set used to track users who have completed the registration process. Since sets only store unique values, it ensures that a user can’t be registered more than once.

- **`user_track_registrations = {}`**: This dictionary tracks the registration of users based on the tracks they select. It maps the user’s ID to the track they are registered in, which helps in organizing and retrieving user information based on their selected tracks.

- **`matched_members = set()`**: This set tracks the users who have already been matched with a leader and have completed the team formation process. This prevents users from being matched more than once.

- **`leader_departments = {}`**: This dictionary stores the department and university information for each leader. This is important to ensure that leaders and members belong to the same university and department for effective collaboration.

---

These data structures are crucial for organizing the registration and team matching process. The bot relies on them to store, retrieve, and update information about users (both leaders and members), ensuring the matching process runs smoothly.

---


#### 7. **Data Storage Functions: `save_data` and `load_data` 💾**

This section contains two important functions: `save_data` and `load_data`. These functions handle the saving and loading of the bot's state, which is crucial for persistence across restarts. When the bot is restarted, it needs to retain user registrations, matches, and other relevant information.
```python
# Data storage functions
def save_data():
    try:
        # Convert member tuples to serializable format
        serialized_members = {}
        for track, member_list in members.items():
            serialized_members[track] = []
            for rating, reg_time, member in member_list:
                # Store user ID and name instead of User object
                serialized_member = {
                    'user_id': member.get('user_id') or member['user'].id,  # Handle both new and old format
                    'user_name': member.get('user_name') or member['user'].name,  # Handle both new and old format
                    'track': member['track'],
                    'rating': member['rating'],
                    'comment': member['comment'],
                    'department': member['department'],
                    'university': member['university'],
                    'selected_topics': member['selected_topics'],
                    'registration_time': member['registration_time']
                }
                serialized_members[track].append((rating, reg_time, serialized_member))

        # Convert leaders to serializable format
        serialized_leaders = {}
        for track, leader_list in leaders.items():
            serialized_leaders[track] = []
            for leader in leader_list:
                # Store user ID and name instead of User object
                serialized_leader = {
                    'user_id': leader.get('user_id') or leader['user'].id,  # Handle both new and old format
                    'user_name': leader.get('user_name') or leader['user'].name,  # Handle both new and old format
                    'team_name': leader['team_name'],
                    'track': leader['track'],
                    'team_comment': leader['team_comment'],
                    'department': leader['department'],
                    'university': leader['university']
                }
                serialized_leaders[track].append(serialized_leader)

        data = {
            'members': serialized_members,
            'leaders': serialized_leaders,
            'user_data': user_data,
            'registered_users': list(registered_users),
            'matched_members': list(matched_members),
            'leader_departments': leader_departments
        }
        
        with open('bot_data.json', 'w') as f:
            json.dump(data, f, default=str)
        logger.info("Data saved successfully")

    except Exception as e:
        logger.error(f"Error saving data: {str(e)}")

def load_data():
    try:
        if os.path.exists('bot_data.json'):
            with open('bot_data.json', 'r') as f:
                data = json.load(f)
            
            # Restore data structures
            global members, leaders, user_data, registered_users, matched_members, leader_departments
            
            # Restore members
            members = defaultdict(list)
            for track, member_list in data['members'].items():
                for rating, reg_time, member_dict in member_list:
                    # Store just the raw data - we'll get the User object when needed
                    member = {
                        'user_id': member_dict['user_id'],
                        'user_name': member_dict['user_name'],
                        'track': member_dict['track'],
                        'rating': member_dict['rating'],
                        'comment': member_dict['comment'],
                        'department': member_dict['department'],
                        'university': member_dict['university'],
                        'selected_topics': member_dict['selected_topics'],
                        'registration_time': member_dict['registration_time']
                    }
                    members[track].append((rating, reg_time, member))
            
            # Restore leaders
            leaders = defaultdict(list)
            for track, leader_list in data['leaders'].items():
                for leader_dict in leader_list:
                    # Store just the raw data
                    leader = {
                        'user_id': leader_dict['user_id'],
                        'user_name': leader_dict['user_name'],
                        'team_name': leader_dict['team_name'],
                        'track': leader_dict['track'],
                        'team_comment': leader_dict['team_comment'],
                        'department': leader_dict['department'],
                        'university': leader_dict['university']
                    }
                    leaders[track].append(leader)
            
            user_data = data['user_data']
            registered_users = set(data['registered_users'])
            matched_members = set(data['matched_members'])
            leader_departments = data['leader_departments']
            
            logger.info("Data loaded successfully")
    except Exception as e:
        logger.error(f"Error loading data: {str(e)}")
```

##### 7.1 **`save_data()`**:
The `save_data` function serializes and saves the bot’s data to a JSON file (`bot_data.json`). Here’s a breakdown:

- **Serializing Members and Leaders**: 
  - The `members` and `leaders` dictionaries are iterated over. Each member and leader is converted into a serializable format that can be saved to a JSON file.
  - This involves extracting relevant details from the user data (e.g., `user_id`, `user_name`, `track`, `rating`, etc.) and storing them in a format that can be easily reconstructed later.
  - For example, the `members` dictionary contains user details, their ratings, and registration times, all of which need to be serialized so they can be saved to the JSON file.

- **Saving Data**: 
  - After serializing the data, it is stored in a dictionary called `data`. This dictionary includes:
    - `members`: The serialized members data.
    - `leaders`: The serialized leaders data.
    - `user_data`: The overall user data (role, track, university, etc.).
    - `registered_users`: A list of user IDs that have registered.
    - `matched_members`: A list of matched members who have already been paired with a leader.
    - `leader_departments`: A dictionary of leaders with their respective departments and universities.

  - **Writing to JSON**: 
    - The `json.dump()` function is used to write the `data` dictionary to a file called `bot_data.json`. The `default=str` argument ensures that all data is properly serialized, even if it’s not directly JSON serializable (e.g., datetime objects).

- **Error Handling**: 
  - A `try-except` block ensures that any errors during the saving process are caught and logged. If saving fails, an error message is logged.

##### 7.2 **`load_data()`**:
The `load_data` function loads the saved data from the JSON file and deserializes it into the appropriate data structures.

- **Checking for File Existence**: 
  - The function first checks whether the `bot_data.json` file exists. If the file is found, it is opened and loaded into memory.

- **Deserialization**:
  - The data loaded from the JSON file is iterated over, and the members and leaders are restored into their original form (with all their attributes).
  - For each user, their track, university, department, topics, ratings, and other information are restored so that they can continue their registration or matching process.
  
- **Restoring Data**:
  - After deserializing the data, the function restores the following:
    - `members`: A list of members, with their corresponding details.
    - `leaders`: A list of leaders, with their team details.
    - `user_data`: A dictionary of all user-related data.
    - `registered_users`: A set of all registered users.
    - `matched_members`: A set of all matched members.
    - `leader_departments`: A dictionary containing the department and university for each leader.

- **Error Handling**:
  - If an error occurs during the loading process (e.g., if the file is corrupted or missing data), an error is logged.

---

These functions provide the bot with persistence, ensuring that data is saved between bot restarts and that users' progress is not lost. They also allow the bot to resume its operation smoothly, with all registered users and matched teams loaded from the previous session.

---


#### 8. **Automatic Restart Function and Error Handling 🛠️**

This section contains an automatic restart function and an error-handling decorator. Both are essential for ensuring the bot runs continuously and handles any unexpected issues gracefully.
```python
# Automatic restart function
def schedule_restart():
    save_data()
    logger.info("Initiating scheduled restart")
    
    python_executable = sys.executable  # Get Python executable path
    script_path = os.path.abspath(sys.argv[0])  # Get absolute path of script

    # Ensure proper quoting of file paths (handles spaces correctly)
    command = [python_executable, script_path]
    os.execv(python_executable, command)

# Error handling decorator
def error_handler(func):
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except Exception as e:
            ctx = args[0] if args else None
            error_msg = f"Error in {func.__name__}: {str(e)}"
            logger.error(error_msg)
            if ctx:
                await ctx.send(f"An error occurred. Please try again or contact support.\nError: {str(e)}")
    return wrapper
```

##### 8.1 **`schedule_restart()`**:
The `schedule_restart` function is responsible for ensuring the bot restarts periodically and safely. Here’s a detailed breakdown:

- **Saving Data**: 
  - Before restarting, the `save_data()` function is called. This ensures that any data (such as user registrations or matches) is saved before the bot restarts.
  
- **Restarting the Bot**:
  - **`python_executable = sys.executable`**: This retrieves the path of the Python interpreter used to run the bot. It ensures that the correct Python environment is used to restart the bot.
  - **`script_path = os.path.abspath(sys.argv[0])`**: This gets the absolute path of the script that is running (i.e., the bot script). It’s used to restart the bot from the exact same script.
  
- **Executing the Restart**: 
  - The command **`os.execv(python_executable, command)`** is used to restart the script. It terminates the current process and starts a new one with the same Python executable and script. This allows the bot to restart automatically at regular intervals.

- **Logging**:
  - A message is logged to indicate that the restart process has started. This provides visibility for administrators monitoring the bot’s activity.

##### 8.2 **`error_handler(func)`**:
The `error_handler` function is a decorator used to handle exceptions that occur during the execution of bot commands.

- **Decorator Function**:
  - A decorator is a special type of function that modifies the behavior of other functions. In this case, `error_handler` is used to wrap other bot commands, ensuring that any errors that occur are caught and handled in a standardized way.
  
- **Error Handling**:
  - **`try-except` block**: The decorator wraps the original function (passed as `func`). If an error occurs during the execution of the wrapped function, it is caught in the `except` block.
  
- **Error Logging**:
  - If an error is caught, an error message is logged with the details of the function where the error occurred (`func.__name__`) and the error message itself (`str(e)`).
  
- **User Notification**:
  - The error is sent to the user via a direct message (`ctx.send`). This notifies the user that an error has occurred and encourages them to try again or contact support.
  
- **Custom Error Responses**:
  - The decorator allows for easy customization of error responses for all commands that use it. This ensures that users always receive helpful error messages in a consistent format.


---

Both the automatic restart function and error handler ensure that the bot operates reliably and can recover from unexpected issues, providing a smooth experience for users.

---

#### 9. **Commands: Registration and Role Selection 📝**

This section defines several bot commands that allow users to interact with the bot, register their information, and select their roles. Each command is decorated with `@bot.command`, which makes it a part of the bot’s command set, allowing users to invoke them by typing `!command_name` in Discord.
```python
# Use these in your command responses, for example:
@bot.command(name="helpbot")
async def helpbot(ctx):
    help_title = "Discord Team Matching Bot Commands"
    help_description = (
        "Welcome to the Team Matching Bot! Here are the available commands:"
    )
    
    fields = [
        ("🔹 Registration Process", "`!start` - Complete entire registration process", False),
        ("🔹 Commands", 
         "`!choose_department` - Select department\n"
         "`!choose_role` - Choose Leader/Member\n"
         "`!choose_track` - Select track\n"
         "`!choose_topics` - Select topics\n"
         "`!write_comment` - Add comment", False),
        ("📌 Registration Flow",
         "1. Choose Department\n"
         "2. Choose Role\n"
         "3. Select Track Category\n"
         "4. Select Specific Track\n"
         "5. Choose Topics\n"
         "6. Write Comment", False)
    ]
    
    embed = format_embed_message(help_title, help_description, fields)
    await ctx.send(embed=embed)

# Update the departments in choose_department command
@bot.command(name="choose_department")
async def choose_department(ctx):
    user_id = ctx.author.id
    role = user_data.get(user_id, {}).get('role')
    university = user_data.get(user_id, {}).get('university')

    # Check if the user is a leader and already has registered before
    if role == "leader" and user_id in leader_departments:
        stored_dept = leader_departments[user_id]['department']
        stored_univ = leader_departments[user_id]['university']
        
        # If university doesn't match previous registration
        if university != stored_univ:
            await ctx.send(f"You must register with your original university: {stored_univ}")
            user_data[user_id]["university"] = stored_univ
            await ctx.invoke(bot.get_command("choose_department"))
            return
            
        # Use the stored department and prevent changes
        user_data[user_id]["department"] = stored_dept
        await ctx.send(f"Your department is already set to {stored_dept.upper()} and cannot be changed.")
        await ctx.invoke(bot.get_command("choose_role"))
        return

    # If the leader doesn't have a department yet or is a member, allow them to choose
    select = Select(
        placeholder="Choose your department",
        options=[
            discord.SelectOption(label="CS", value="cs"),
            discord.SelectOption(label="IT", value="it"),
            discord.SelectOption(label="IS", value="is"),
            discord.SelectOption(label="AI", value="ai"),
            discord.SelectOption(label="SW", value="sw"),
            discord.SelectOption(label="BIO", value="bio")
        ]
    )

    async def callback(interaction):
        user_data[user_id]["department"] = select.values[0]
        
        # If the user is a leader, store their department and university permanently
        if role == "leader":
            leader_departments[user_id] = {
                'department': select.values[0],
                'university': user_data[user_id].get('university')
            }
        
        await interaction.response.send_message(f"You have chosen {select.values[0].upper()} department.")
        await ctx.invoke(bot.get_command("choose_role"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Please choose your department:", view=view)
```

##### 9.1 **`@bot.command(name="helpbot")`**:
This command provides users with a list of all available commands and instructions on how to use the bot.

- **Command Response**: 
  - The bot sends a help message to the user, detailing the steps for registration and the available commands. This is useful for new users who might not know how to interact with the bot.
  - The help message is formatted using an embed (a rich message format in Discord), which includes sections like:
    - **Registration Process**: Describes how to complete the entire registration process, step by step.
    - **Commands**: Lists all the available commands that users can use, such as `!choose_department` and `!choose_role`.
    - **Registration Flow**: Provides a simplified version of the registration steps to guide users.

- **`embed = format_embed_message(...)`**: 
  - This line creates the formatted embed message that will be sent to the user. The `format_embed_message` function (which is defined later) is used to generate the structure of the help message, including its title, description, and fields.

##### 9.2 **`@bot.command(name="choose_department")`**:
This command handles the department selection process during the registration phase. 

- **Department Selection**:
  - When a user calls this command, they are prompted to select their department (e.g., `CS`, `IT`, `AI`). 
  - The selection is made using a `Select` menu, which provides a user-friendly dropdown interface in Discord. 
  - If the user has already registered as a leader, the bot checks if their department matches their previous registration. If there’s a mismatch, it prompts the user to select the correct department.
  
- **Leader-Specific Logic**:
  - If the user is a leader and has already selected their department in a previous session, the bot ensures that their university and department match the original data stored in `leader_departments`. This prevents leaders from changing their department or university after registration.

- **Callback for Selection**:
  - The `Select` menu has a callback function, `callback(interaction)`, which is called when the user selects an option. This callback updates the user’s department in `user_data` and proceeds to the next step in the registration process (asking them to select their role).

- **User Feedback**:
  - After the user selects a department, the bot sends a confirmation message to the user and automatically moves them to the next step (`choose_role`).


---

These registration commands help guide the user through the bot’s setup, ensuring that they select their department and role correctly before proceeding to the next steps. The commands are structured to be intuitive and responsive to users' previous inputs, such as storing their department and university for future reference.

---

#### 10. **Role Selection Command: `choose_role` 🧑‍💻**

The `choose_role` command allows users to select their role as either a **Leader** or a **Member**. This is a crucial step in the registration process, as it determines the user’s responsibility in the team formation process.
```python
# Role Selection Command
@bot.command(name="choose_role")
async def choose_role(ctx):
    user_id = ctx.author.id
    
    select = Select(
        placeholder="Choose your role",
        options=[
            discord.SelectOption(label="Leader", value="leader"),
            discord.SelectOption(label="Member", value="member")
        ]
    )

    async def callback(interaction):
        selected_role = select.values[0]
        
        # Check if user is choosing to be a leader and has registered before
        if selected_role == "leader" and user_id in leader_departments:
            stored_univ = leader_departments[user_id]['university']
            stored_dept = leader_departments[user_id]['department']
            current_univ = user_data[user_id].get('university')
            current_dept = user_data[user_id].get('department')
            
            # Verify university and department match previous registration
            if current_univ != stored_univ or current_dept != stored_dept:
                await interaction.response.send_message(
                    f"⚠️ You must register with your original university ({stored_univ}) "
                    f"and department ({stored_dept.upper()}). Please start over with !start"
                )
                return
        
        user_data[user_id]["role"] = selected_role
        
        if selected_role == "leader":
            await interaction.response.send_message("You have chosen to be a Leader.")
            await ctx.send("Please enter your team name:")
            
            def check(m):
                return m.author == ctx.author and m.channel == ctx.channel
            
            try:
                msg = await bot.wait_for("message", check=check, timeout=60)
                team_name = msg.content
                user_data[user_id]["team_name"] = team_name
                await ctx.send(f"Your team name is {team_name}.")
                
                # Store leader's information if this is their first registration
                if user_id not in leader_departments:
                    leader_departments[user_id] = {
                        'university': user_data[user_id]['university'],
                        'department': user_data[user_id]['department']
                    }
                
                await ctx.invoke(bot.get_command("choose_track"))
            except asyncio.TimeoutError:
                await ctx.send("Team name selection timed out. Please try again.")
        
        else:  # Member
            await interaction.response.send_message("You have chosen to be a Member.")
            await ctx.invoke(bot.get_command("choose_track"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Choose your role:", view=view)
```

##### 10.1 **Role Selection Logic**:
- **Role Options**:
  - The user is presented with two role options: **Leader** or **Member**, via a `Select` dropdown menu. The `Select` menu lets users choose one of the options easily.
  - **Leaders** are responsible for creating a team, specifying project ideas, and managing the team. They will proceed to create and submit a team name after selecting their role.
  - **Members** will be assigned to a team led by a leader and will contribute to the project according to their skills.

- **Role Verification for Leaders**:
  - If the user selects the **Leader** role, the bot checks if the user has already registered as a leader and whether their department and university match the stored values. This prevents a user from registering as a leader in a different department or university.
  - If the department and university don’t match, the bot will prompt the user to start the registration process over using `!start`.

- **Callback Function for Role Selection**:
  - When the user selects their role, the `callback(interaction)` function is triggered. This function updates the user's role in the `user_data` dictionary and proceeds with the next steps in the registration process:
    - If the user is a **Leader**, the bot asks them to provide their **team name**.
    - If the user is a **Member**, the bot moves them directly to the next step, where they will choose their **track**.

- **Team Name for Leaders**:
  - Once a leader selects their role, the bot asks for their **team name**. The bot waits for the leader to send the team name as a message, using `bot.wait_for("message")` to capture the response.
  - The bot stores the **team name** in `user_data` for future reference and prompts the leader to continue to the next step (choosing their track).

- **Handling Timeouts**:
  - If the leader doesn’t provide a team name within a specified time (60 seconds in this case), the bot will time out and prompt the user to try again.

- **Moving to the Next Step**:
  - After the user has selected their role (and, if applicable, provided their team name), the bot invokes the `choose_track` command to move the user to the next step in the registration process. This ensures that the user completes all the necessary steps in sequence.

---

This command is essential for guiding users to select their role, ensuring the process is smooth and accurate. It is important for leaders to be identified early, as they are the ones who will manage teams, while members will be assigned accordingly.

---

#### 11. **Track Selection Command: `choose_track` 🔧**

The `choose_track` command is a critical part of the registration process where users select their preferred track or specialization. This is the stage where the users (both leaders and members) choose what area they want to work on in their graduation project. Tracks are categorized into different areas like Backend Frameworks, Cybersecurity, Data Science, etc.
```python
# Track Selection Command
@bot.command(name="choose_track")
async def choose_track(ctx):
    role = user_data.get(ctx.author.id, {}).get('role')
    
    select = Select(
        placeholder="Choose track category",
        options=[discord.SelectOption(label=category, value=category) for category in track_categories.keys()]
    )

    async def category_callback(interaction):
        category = select.values[0]
        track_options = [
            discord.SelectOption(label=track, value=track) 
            for track in track_categories[category]
        ]
        
        track_select = Select(
            placeholder=f"Choose track in {category}",
            options=track_options
        )

        async def track_callback(track_interaction):
            selected_track = track_select.values[0]
            
            # Remove previous track restrictions for leaders
            user_data[ctx.author.id]['track'] = selected_track
            await track_interaction.response.send_message(f"You have chosen {selected_track}.")
            
            # Different flow for leader and member
            if role == "leader":
                await ctx.invoke(bot.get_command("write_comment"))
            else:
                await ctx.invoke(bot.get_command("choose_topics"))

        track_select.callback = track_callback
        track_view = View()
        track_view.add_item(track_select)
        await interaction.response.send_message(f"Choose a track in {category}:", view=track_view)

    select.callback = category_callback
    view = View()
    view.add_item(select)
    await ctx.send("Choose a track category:", view=view)
```

##### 11.1 **Track Category Selection**:
- **Track Categories**:
  - Users are first prompted to choose a **track category** (e.g., Backend Frameworks, Cybersecurity, Data Science). A `Select` dropdown menu is used to display the track categories available.
  - This initial selection simplifies the process by grouping related tracks together, making it easier for users to find their area of interest.

- **Handling Track Options**:
  - Once a track category is chosen, the bot dynamically generates a new set of options based on the selected category. For example, if a user chooses "Backend Frameworks", they will see track options like `.NET`, `Node.js`, `Laravel`, etc.
  - This dynamic generation ensures that the bot only shows relevant tracks based on the user’s category selection.

- **Callback Function for Track Selection**:
  - The callback function for this `Select` menu is triggered when the user selects a specific track. This function updates the user's track choice in the `user_data` dictionary.
  - After selecting the track, the bot will send a confirmation message and then proceed to the next step based on the user's role:
    - If the user is a **Leader**, the bot proceeds to the `write_comment` command, where the leader will write a comment about the team and project.
    - If the user is a **Member**, the bot moves them to the `choose_topics` command, where they can select topics related to their track.

- **Role-Specific Flow**:
  - The flow diverges based on whether the user is a leader or a member:
    - **Leaders**: They need to provide more details about their project and team, which is why they are prompted for a comment about their team after selecting the track.
    - **Members**: They are prompted to select topics related to the track they've chosen, which helps the bot assess their skills for matching with a leader later.

---

This command is crucial for allowing users to define the area of expertise they want to focus on for their graduation project. By selecting a track, the bot can ensure that users are matched with others who have similar interests and technical skills, increasing the likelihood of successful project collaboration.

---


#### 12. **Topic Selection Command: `choose_topics` 🧠**

The `choose_topics` command is where users (both leaders and members) select the topics they have studied and are knowledgeable about within their chosen track. These topics will help assess the user's skill level and assist the bot in matching them with others in the project.
```python
# Topic Selection Command
@bot.command(name="choose_topics")
async def choose_topics(ctx):
    track = user_data[ctx.author.id]["track"]
    topics = [topic['name'] for topic in track_topics[track]]
    
    select = Select(
        placeholder="Choose topics you've studied",
        options=[discord.SelectOption(label=topic, value=topic) for topic in topics],
        max_values=len(topics)
    )

    async def callback(interaction):
        selected_topics = select.values
        user_data[ctx.author.id]["selected_topics"] = selected_topics
        rating = calculate_rating(ctx.author.id)
        user_data[ctx.author.id]["rating"] = rating
        
        await interaction.response.send_message(f"You have selected: {', '.join(selected_topics)}. Your rating is {rating}%.")
        await ctx.invoke(bot.get_command("write_comment"))

    select.callback = callback
    view = View()
    view.add_item(select)
    await ctx.send("Please select topics you've studied:", view=view)
```

##### 12.1 **Topic Selection Process**:
- **Topic List**:
  - After selecting a track, users are presented with a list of topics related to that track. For instance, if a user selects a **Backend Framework** like `.NET`, the bot will present topics like `C# Basics`, `.NET Core Fundamentals`, and others in that category.
  
- **Selecting Topics**:
  - The users can select multiple topics they are familiar with by using the `Select` dropdown menu. The `max_values` parameter is set to the maximum number of topics available for the selected track, ensuring that users can select as many topics as they want.
  
- **Updating User Data**:
  - Once the topics are selected, the bot updates the `user_data` dictionary with the chosen topics. This data is essential for calculating the user's **rating**, which is later used in the matching algorithm to pair leaders and members.
  
- **Rating Calculation**:
  - The `rating` is calculated based on the topics selected by the user. Each topic has a predefined score (e.g., 15, 25, or 40 points depending on its difficulty). The total score of the selected topics is used to determine the user's **rating**.
  - This rating is stored in `user_data` and will play a role in determining the user's compatibility with other users during the matching process.
  
- **Callback Function for Topic Selection**:
  - When the user selects their topics, the `callback(interaction)` function is triggered. This function:
    1. Updates the user's selected topics in `user_data`.
    2. Calculates the user's rating based on the selected topics.
    3. Sends a message confirming the user's selected topics and rating.
    4. Proceeds to the next step in the registration process:
      - **For Members**: They are then prompted to write a comment about themselves.
      - **For Leaders**: They are asked to provide a comment about their team.

- **User Feedback**:
  - After the topics are selected, the bot sends a confirmation message to the user, displaying the topics they have chosen and their rating.

---

This command plays a critical role in assessing users' knowledge and skills. By selecting topics, users help the bot evaluate their expertise, and the bot uses this information to match them with others who have complementary skills, ensuring better project collaboration.

---


#### 13. **Comment Writing Command: `write_comment` ✍️**

The `write_comment` command prompts the user to provide a comment about themselves, their team (if they are a leader), and their project. This is the final step in the registration process and serves as a way for users to describe their project idea, their role, and their expectations.
```python
# Write Comment Command
@bot.command(name="write_comment")
async def write_comment(ctx):
    role = user_data[ctx.author.id]["role"]
    
    if role == "member" and ctx.author.id in registered_users:
        await ctx.send("You have already registered. Wait for matching.")
        return

    comment_prompt = (
        "Write A Comment About Your Team Like Members In It And Your Project Idea.." 
        if role == "leader" else 
        "Write A Comment About Yourself Like What You Have Studied And What You Will Study In The Future.."
    )
    
    await ctx.send(comment_prompt)

    def check(m):
        return m.author == ctx.author and m.channel == ctx.channel

    try:
        msg = await bot.wait_for("message", check=check, timeout=60)
        comment = msg.content
        user_data[ctx.author.id]["comment"] = comment
        registration_time = time.time()
        
        track = user_data[ctx.author.id]["track"]
        department = user_data[ctx.author.id]["department"]
        university = user_data[ctx.author.id]["university"]
        
        if role == "member":
            member = {
                "user_id": ctx.author.id,  # Store ID instead of User object
                "user_name": ctx.author.name,  # Store name separately
                "track": track,
                "rating": user_data[ctx.author.id]["rating"],
                "comment": comment,
                "department": department,
                "university": university,
                "selected_topics": user_data[ctx.author.id].get("selected_topics", []),
                "registration_time": registration_time
            }
            heapq.heappush(members[track], (-member["rating"], registration_time, member))
            registered_users.add(ctx.author.id)
        else:
            leader = {
                "user_id": ctx.author.id,  # Store ID instead of User object
                "user_name": ctx.author.name,  # Store name separately
                "team_name": user_data[ctx.author.id]["team_name"],
                "track": track,
                "team_comment": comment,
                "department": department,
                "university": university
            }
            leaders[track].append(leader)
        
        await ctx.send("Your registration is complete. You've been added to the matching queue.")
        await ctx.invoke(bot.get_command("match"))
        
    except asyncio.TimeoutError:
        await ctx.send("Comment submission timed out. Please try again.")
```

##### 13.1 **Comment Writing Logic**:
- **Role-Based Prompt**:
  - **Leaders** are asked to write a comment about their team, including team members and the project idea. This helps the bot understand the leader’s vision for the project and provides context for matching members.
  - **Members** are asked to write a comment about themselves, such as what they have studied, what topics they will study in the future, and how they envision contributing to the project.
  
- **Message Handling**:
  - The bot waits for the user’s response using `bot.wait_for("message")`. This function listens for a message from the user in the same channel and stores that message as the user’s comment.
  - If the user doesn’t provide a response within the timeout period (60 seconds), the bot will time out and prompt the user to try again. This helps ensure that the registration process continues smoothly without unnecessary delays.
  
- **Storing the Comment**:
  - Once the comment is received, the bot stores it in `user_data[ctx.author.id]["comment"]`. This allows the bot to keep track of the user's input for future reference, especially when generating team match profiles.
  
- **User Registration**:
  - **For Members**: The member’s information is stored in the `members` data structure. A new member is added with their track, rating, comment, and other registration details. They are then added to the `registered_users` set to track who has completed registration.
  - **For Leaders**: The leader’s information, including their team name and comments, is stored in the `leaders` data structure. The leader is then added to the matching process, where they will be paired with members who match their project idea.
  
- **Proceeding to Matching**:
  - After the comment is written and the data is stored, the bot proceeds to the next step in the process:
    - **For Members**: The bot sends a message confirming that the member has completed their registration and that they have been added to the matching queue.
    - **For Leaders**: The bot sends a similar confirmation and proceeds with the matching process once the registration is complete.

---

This command is important for capturing the user's vision (if they are a leader) or personal details (if they are a member). The comments are a valuable part of the matching algorithm, as they provide insights into the users’ motivations, team goals, and expectations. By collecting this information, the bot ensures that the matching process is as smooth and tailored as possible.

---


#### 14. **Start Command: `start` 🏁**

The `start` command is the first step in the registration process. It is responsible for initializing the user's registration, ensuring that they are guided through the process of selecting their university, department, role, track, and topics.
```python
# Update the start command to begin with university selection
# Add error handling to existing commands
@bot.command(name="start")
@error_handler
@commands.cooldown(1, 15, BucketType.user)
async def start(ctx):
    user_id = ctx.author.id
    
    # If user is already registered as a leader, verify their status
    if user_id in leader_departments:
        stored_data = leader_departments[user_id]
        await ctx.send(
            f"⚠️ You have previously registered as a leader for:\n"
            f"University: {stored_data['university']}\n"
            f"Department: {stored_data['department'].upper()}\n\n"
            f"You must use these same details to register again."
        )
    
    # Check if the member is already registered and not matched
    if user_id in registered_users:
        await ctx.send("You have already registered. Wait for matching.")
        return

    # Clear any existing data for the user
    if user_id in user_data:
        del user_data[user_id]
    
    await ctx.invoke(bot.get_command("choose_university"))
    save_data()
```

###### 14.1 **Initial Setup**:
- **Checking Previous Registration**:
  - The command starts by checking if the user has already completed registration and is waiting for matching. If the user is already registered (either as a leader or a member), the bot will send a message indicating that the user must wait for their matching to be processed.
  - If the user has previously registered as a **Leader**, the bot verifies whether the university and department they’ve selected match their previous registration. If there is a mismatch, the bot will prompt the user to start over with `!start`, ensuring consistency in the data.

##### 14.2 **Handling Users Who Need to Start Registration**:
- **Clearing Old Data**:
  - If the user is starting the registration process from scratch (or if the previous data has been cleared), the bot will delete any old registration data stored in `user_data`. This ensures that the new registration is clean and accurate.
  
- **Proceeding to the Next Step**:
  - Once any previous registration issues are handled, the bot prompts the user to select their **university** by invoking the `choose_university` command.
  - The `save_data()` function is called at the end of the command to ensure that the bot’s data is saved at each important step. This allows the bot to persist user information even after a restart.

##### 14.3 **Why is this Command Important?**:
- **Kickstarting the Registration**:
  - The `start` command acts as the gateway to the registration process, ensuring that users can start the registration sequence correctly, either from scratch or by verifying their previous information.
  
- **Consistency in Data**:
  - By verifying the user's university and department, the `start` command ensures that the registration process is consistent and that leaders cannot change important details after their initial registration.

---

This command serves as the entry point to the user registration process, allowing the bot to guide users through the necessary steps, and ensuring that their details are consistent throughout the process.

---


#### 15. **Perform Matching Process: `perform_matching` 🤝**

The `perform_matching` function is the core of the team matching process. It uses the information collected during registration (like users’ roles, tracks, topics, and ratings) to match **leaders** and **members** based on their compatibility.
```python
#Perform_Matching Process Function
async def perform_matching():
    try:
        matching_track_count = 0
        
        for category, tracks in track_categories.items():
            for track in tracks:
                track_leaders = leaders.get(track, []).copy()
                track_members = members.get(track, [])
                
                while track_leaders and track_members:
                    try:
                        leader = track_leaders[0]  # Don't pop yet, wait until match is confirmed
                        matching_member = None
                        temp_members = []
                        
                        while track_members:
                            rating, reg_time, member = heapq.heappop(track_members)
                            
                            # Check if member is still in registered_users
                            if (member['user_id'] in registered_users and 
                                leader["university"] == member["university"] and 
                                leader["department"] == member["department"]):
                                matching_member = member
                                break
                            else:
                                temp_members.append((rating, reg_time, member))
                        
                        # Restore unmatched members back to the heap
                        for m in temp_members:
                            heapq.heappush(track_members, m)
                        
                        if not matching_member:
                            track_leaders.pop(0)  # Remove leader if no match found
                            continue

                        # Get the actual Discord User objects
                        try:
                            leader_user = await bot.fetch_user(leader['user_id'])
                            member_user = await bot.fetch_user(matching_member['user_id'])
                            
                            if not leader_user or not member_user:
                                logger.error("Could not fetch user objects")
                                continue
                                
                            leader_channel = await leader_user.create_dm()
                            member_channel = await member_user.create_dm()
                        except discord.HTTPException as e:
                            logger.error(f"Failed to create DM channels: {e}")
                            continue

                        # Format messages
                        success_header = "```ansi\n\u001b[1;32m🎉 MATCHING SUCCESS! 🎉\u001b[0m\n```"
                        
                        leader_match_info = (
                            f"{success_header}\n"
                            f"**📋 Match Details**\n"
                            f"```yml\n"
                            f"Member Name   : {member_user.name}\n"
                            f"University    : {matching_member['university']}\n"
                            f"Department    : {matching_member['department'].upper()}\n"
                            f"Track         : {matching_member['track']}\n"
                            f"Team          : {leader['team_name']}\n"
                            f"```\n"
                        )

                        member_match_info = (
                            f"{success_header}\n"
                            f"**📋 Match Details**\n"
                            f"```yml\n"
                            f"Team Leader   : {leader_user.name}\n"
                            f"University    : {leader['university']}\n"
                            f"Department    : {leader['department'].upper()}\n"
                            f"Track         : {leader['track']}\n"
                            f"Team          : {leader['team_name']}\n"
                            f"```\n"
                        )

                        team_info = (
                            f"**🏢 Team Information**\n"
                            f"```ansi\n"
                            f"\u001b[1;34m── Team Leader's Message ──\u001b[0m\n"
                            f"{leader['team_comment']}\n"
                            f"```"
                        )

                        topics_str = ", ".join(matching_member["selected_topics"])
                        member_profile = (
                            f"**👤 Member Profile**\n"
                            f"```ansi\n"
                            f"\u001b[1;33m── Technical Background ──\u001b[0m\n"
                            f"• Track: {matching_member['track']}\n"
                            f"• Rating: {matching_member['rating']}%\n\n"
                            f"\u001b[1;33m── Topics Studied ──\u001b[0m\n"
                            f"{topics_str}\n\n"
                            f"\u001b[1;33m── Personal Note ──\u001b[0m\n"
                            f"{matching_member['comment']}\n"
                            f"```"
                        )

                        contact_info = (
                            f"**📱 Next Steps**\n"
                            f"```ansi\n"
                            f"\u001b[1;35mYou can now communicate directly through Discord!\u001b[0m\n"
                            f"Feel free to discuss project details and next steps.\n"
                            f"```"
                        )

                        # Send all messages with error handling
                        try:
                            await leader_channel.send(leader_match_info)
                            await leader_channel.send(team_info)
                            await leader_channel.send(member_profile)
                            await leader_channel.send(contact_info)
                            
                            await member_channel.send(member_match_info)
                            await member_channel.send(team_info)
                            await member_channel.send(member_profile)
                            await member_channel.send(contact_info)
                            
                            # Only proceed with cleanup if messages were sent successfully
                            # Clean up after successful match
                            leader_id = leader['user_id']
                            member_id = matching_member['user_id']
                            
                            track_leaders.pop(0)  # Now safe to remove the leader
                            leaders[track] = [l for l in leaders[track] if l['user_id'] != leader_id]
                            
                            if leader_id in user_data:
                                del user_data[leader_id]
                            if member_id in user_data:
                                del user_data[member_id]
                            
                            registered_users.discard(member_id)
                            matched_members.add(member_id)
                            
                            if leader_id in leader_departments:
                                del leader_departments[leader_id]
                            
                            matching_track_count += 1
                            logger.info(f"Successful match in track {track}: Leader {leader_id} with Member {member_id}")
                            
                        except discord.HTTPException as e:
                            logger.error(f"Failed to send match messages: {e}")
                            continue
                        except Exception as e:
                            logger.error(f"Error during match cleanup: {e}")
                            continue
                        
                    except Exception as e:
                        logger.error(f"Error in matching process for track {track}: {str(e)}")
                        continue
        
        # Save data after all matches are complete
        try:
            save_data()
        except Exception as e:
            logger.error(f"Error saving data after matching: {str(e)}")
        
        return matching_track_count
        
    except Exception as e:
        logger.error(f"Error in perform_matching: {str(e)}")
        return 0
```

##### 15.1 **Matching Logic**:
- **Loop Through Track Categories and Tracks**:
  - The function begins by iterating over each **track category** and the tracks within that category. For each track, it retrieves a list of **leaders** and **members**.
  
- **Matching Leaders and Members**:
  - The matching process proceeds by trying to pair a **leader** with a **member**:
    - For each **leader**, the bot attempts to find a **member** who has:
      - A matching **university** and **department**.
      - A high enough **rating** based on the topics they have selected.
      
- **Priority Queue for Matching**:
  - The bot uses a **heap** (implemented with `heapq`) to store members in order of their **rating**. The heap ensures that members with higher ratings are considered first for matching. If a member has the required rating, they are paired with the leader.
  
- **Matching and Confirmation**:
  - Once a match is found, the bot sends a **direct message** (DM) to both the leader and the member, informing them of the match. The DM includes detailed information about the match, such as:
    - **Leader’s name** and **team name**.
    - **Member’s name**, **rating**, and **selected topics**.
    - The **project idea** and **team member details**.
  
- **Clean-Up**:
  - After a successful match, the leader and the member are removed from the matching queue, and their registration data is cleared.
  - The bot ensures that the matched users are no longer available for matching again and updates the `matched_members` set to track all successfully matched users.

##### 15.2 **Why is this Function Critical?**:
- **Core Matching Logic**:
  - This function is crucial for the success of the entire bot. It ensures that **leaders** are paired with **members** who have the right skills and background, leading to more effective project collaborations.
  
- **Fairness**:
  - The use of the **priority queue** ensures that members with higher ratings are matched first, which helps maintain fairness in the process.

- **Automation**:
  - By automating the matching process, the bot removes the need for manual intervention, saving time and reducing human error.

---

The `perform_matching` function is the heart of the matching process, ensuring that the right people are paired together for successful graduation projects. By matching users based on ratings, department, and university, the bot optimizes team formation and ensures compatibility among users.

---

#### 16. **Match Command: `match` ⚡**

The `match` command is responsible for triggering the **perform_matching** function, which handles the actual team matching process. This command is typically invoked when the bot has gathered all necessary data from users (i.e., their registration information, topics, etc.), and is ready to start pairing leaders with members.
```python
# Match Command with Enhanced Matching and Communication
@bot.command(name="match")
async def match(ctx):
    bot.loop.create_task(perform_matching())
    await ctx.send("Matching process started in the background.")
```

##### 16.1 **Functionality**:
- **Triggering the Matching Process**:
  - The `match` command is invoked manually, either by the bot administrator or through an automated process. Once invoked, it starts the **perform_matching** function in the background.
  - It then sends a message to the channel where the command was issued, indicating that the matching process has started in the background. This message lets users know that the bot is processing their matching requests.

##### 16.2 **Why is this Command Important?**:
- **Starts the Matching Process**:
  - This command acts as the trigger for the entire team-matching process. It ensures that the bot will start the matching process once all necessary data has been gathered from the users.

- **User Notification**:
  - The confirmation message sent to the channel informs users that the process has begun, keeping them in the loop. This is especially important if there are many users or if the matching process takes time.

---

The `match` command is vital for starting the matching process once users are registered. It ties everything together by invoking the function that pairs users based on their track, skills, and rating.

---

#### 17. **Automatic Matching Task: `auto_match` ⏰**

The `auto_match` task is an automated background task that runs every 30 seconds. This task continuously checks for any users who need to be matched and performs the matching process on its own, without requiring manual input from the bot administrators.
```python
# Automatic Matching Task
@tasks.loop(seconds=30)
async def auto_match():
    await perform_matching()
```

##### 17.1 **Functionality**:
- **Automatic Execution**:
  - This task is set up using `@tasks.loop(seconds=30)`, which tells the bot to execute the `perform_matching` function every 30 seconds. It’s an asynchronous task that runs independently of other bot commands, ensuring that the bot is always working on matching users, even if no commands are being manually issued.
  
- **Continuous Matching**:
  - The bot performs the matching process on a regular interval (every 30 seconds), ensuring that as soon as a user finishes registration, they are considered for matching without waiting for an external trigger.
  
- **Why is this Important?**:
  - This background task ensures that the matching process is ongoing, making the bot more efficient and reducing the need for manual intervention. It allows users to be paired automatically, without waiting for someone to manually start the process.


#### 18. **Event Handlers: `on_ready` 🛠️**
The `on_ready` event is triggered when the bot has successfully logged in and is ready to begin interacting with Discord servers. This is the event that ensures the bot is fully operational after startup.
```python
# Start the auto-match task when the bot is ready
# Update the bot's event handlers
@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')
    logger.info(f'Bot logged in as {bot.user}')
    load_data()  # Load saved data
    auto_match.start()
    
    # Schedule restart every hour
    schedule.every(45).minutes.do(schedule_restart)
    
    # Start schedule checker
    while True:
        schedule.run_pending()
        await asyncio.sleep(30)
```

##### 18.1 **Functionality**:
- **Bot Startup**:
  - Once the bot has logged in, the `on_ready` event is triggered, printing a message that confirms the bot is operational. It also logs this event, which is helpful for debugging and monitoring purposes.
  
- **Loading Data**:
  - In this event, the bot calls the `load_data()` function to load any persisted data (such as previous user registrations, matches, and settings). This ensures that when the bot restarts, it can resume from where it left off, maintaining the integrity of the match queue.
  
- **Start Auto-Matching**:
  - After loading data, the `auto_match.start()` function is called to begin the automated matching process. This ensures that as soon as the bot is online, it starts matching users automatically.
  
- **Scheduling a Restart**:
  - The bot uses the `schedule` library to schedule a restart every 45 minutes. This is important for maintaining bot performance and ensuring it operates smoothly without any issues. This function will invoke `schedule_restart()`, which saves the bot's data and restarts the bot.


The `on_command_error` event handler is responsible for handling errors that occur when a command is invoked. This helps prevent the bot from crashing and ensures that users receive helpful error messages.
```python
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandNotFound):
        await ctx.send("```❌ Invalid command. Use `!helpbot` for a list of commands.```")
    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send("```❌ Missing required argument. Please check the command usage.```")
    elif isinstance(error, commands.CommandOnCooldown):
        await ctx.send(f"```⏳ Please wait {error.retry_after:.2f}s before using this command again.```")
    else:
        error_msg = f"Command error: {str(error)}"
        logger.error(error_msg)
        await ctx.send(f"```❌ An error occurred. Please try again later.\nError: {str(error)}```")
```


##### 18.2 **Functionality**:
- **Command Not Found**:
  - If the bot encounters a command that it doesn't recognize, it sends a message to the user indicating that the command is invalid and provides instructions on how to use the bot (`!helpbot`).
  
- **Missing Arguments**:
  - If a user tries to invoke a command without providing the necessary arguments, the bot will inform them that they missed an argument and provide guidance on how to properly use the command.
  
- **Command on Cooldown**:
  - If a user tries to use a command too frequently, the bot will send a message telling them to wait before trying again, preventing spam and overloading the system.

- **General Errors**:
  - For any unexpected errors, the bot logs the error and sends a generic error message to the user, ensuring that the user experience is smooth even when things go wrong.


#### 19. **Main Function: `main` 🧑‍💻**

The `main` function is where the bot is started and handles the graceful shutdown process. This function ensures that the bot operates smoothly and that all necessary resources are cleaned up when the bot shuts down.
```python
# Run the bot
# write this if you use terminal or cmd : set DISCORD_BOT_TOKEN=your_bot_token_here
# write this if you use powershell : $env:DISCORD_BOT_TOKEN="your_bot_token_here"
# Update the main function
async def main():
    try:
        # Handle shutdown gracefully
        def signal_handler(sig, frame):
            logger.info("Shutdown signal received")
            save_data()
            sys.exit(0)
        
        signal.signal(signal.SIGINT, signal_handler)
        signal.signal(signal.SIGTERM, signal_handler)
        
        async with bot:
            await bot.start(os.getenv("DISCORD_BOT_TOKEN"))
    except Exception as e:
        logger.error(f"Main function error: {str(e)}")
        sys.exit(1)

if __name__ == "__main__":
    asyncio.run(main())
```

##### 19.1 **Functionality**:
- **Signal Handling**:
  - The bot listens for shutdown signals (such as `SIGINT` and `SIGTERM`) so that it can save all the data and close gracefully before exiting.
  
- **Starting the Bot**:
  - The bot is started by calling `await bot.start(os.getenv("DISCORD_BOT_TOKEN"))`. The bot token is securely retrieved from the environment variables to avoid exposing sensitive data.
  
- **Graceful Shutdown**:
  - The bot handles shutdowns by calling the `signal_handler()` function, which ensures that the bot’s data is saved before exiting. This prevents data loss if the bot needs to be restarted or shut down for any reason.


## Conclusion and Summary 🎓🤖

The **Graduation Team Matching Discord Bot** represents a breakthrough in automating the process of forming teams for university graduation projects. With its sophisticated matching algorithm, the bot simplifies the typically time-consuming and error-prone task of pairing team members and leaders based on their skills, interests, and academic background. By categorizing users into leaders and members, allowing them to select their preferred tracks and topics, and ensuring alignment in terms of university and department, the bot guarantees the formation of well-suited teams that can collaborate effectively on their projects.

Key features such as role-based registration, automated matching, real-time notifications, and data persistence ensure that the entire process is seamless, efficient, and transparent. The bot’s error handling mechanisms further contribute to a smooth user experience, providing helpful feedback and maintaining the integrity of the registration process.

By running the matching algorithm continuously and offering an intuitive set of commands, the bot saves users valuable time and effort, enabling them to focus on the important aspects of their projects rather than the complexities of team formation. The inclusion of a robust logging and data management system ensures that even after bot restarts, all crucial data is preserved, allowing the matching process to continue uninterrupted.

Ultimately, the **Graduation Team Matching Discord Bot** is not just a tool, but an essential asset for students and academic institutions. It embodies the future of efficient team collaboration, offering a user-friendly solution for students across various disciplines and universities. With its ability to scale, automate, and ensure fairness, the bot is set to revolutionize the way graduation project teams are formed, making collaboration smoother, fairer, and more aligned with project goals.

As educational environments become increasingly digital, tools like this bot pave the way for more innovative and collaborative approaches to learning, teamwork, and academic achievement. By leveraging technology to streamline complex processes, we can unlock new levels of efficiency and foster deeper connections among students, ultimately leading to more successful outcomes in academic and professional pursuits.

---

We hope this guide has been helpful in understanding how the **Graduation Team Matching Discord Bot** works. If you’re looking to implement this in your academic setting or would like to customize it further for your specific needs, the code and documentation are available for review and improvement. Stay tuned for future updates as we continue to refine and expand the bot’s features to meet the evolving demands of education and teamwork.
