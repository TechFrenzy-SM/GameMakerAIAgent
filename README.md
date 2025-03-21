# Game Maker Agent

This project is about creating video games using AI Agents. The framework used here is Autogen 0.2. The LLM model used is GPT-4o hosted on Azure AI Foundry. The code is written in Python3.

| Framework Name | Model Used | Platform         | Language |
| -------------- | ---------- | ---------------- | -------- |
| Autogen 0.2    | GPT-4o     | Azure AI Foundry | Python3  |

## Project Overview

This project leverages AI agents to automate the creation of video games. The agents are designed to handle various aspects of game development, from design to implementation.

## Code Overview

The code is organized into several modules, each responsible for a specific part of the game creation process. The main agents involved in this project are:

1. **User Proxy Agent**: This agent acts as an intermediary between the user and the other agents. It executes the code written by the coder and suggests updates if there are errors.

   ```python
   user_proxy = autogen.UserProxyAgent(
       name="User",
       system_message="Executor. Execute the code written by the coder and suggest updates if there are errors.",
       human_input_mode="NEVER",
       code_execution_config={
           "last_n_messages": 3,
           "work_dir": "code",
           "use_docker": False,
       },
   )
   ```

2. **Coder Agent**: This agent is responsible for writing the game code. It uses the GPT-4o model to generate complete code based on the design specifications and saves the code to disk.

   ```python
   coder = autogen.AssistantAgent(
       name="Coder",
       llm_config=llm_config,
       system_message="""Coder. Your job is to write complete code. You primarily are a game programmer. Make sure to save the code to disk in the work directory.
                         If you want the user to save the code in a file before executing it, put # filename: <filename> inside the code block as the first line.
                         """,
   )
   ```

3. **Product Manager Agent**: This agent helps plan out the game creation process. It provides guidance on how to structure the code and focuses on the requirements mentioned by the user.

   ```python
   pm = autogen.AssistantAgent(
       name="Product_manager",
       system_message="Help plan out to create games. If should not write the code, but can provide guidance on how to structure the code and the main focus should be on the requirements mentioned by the user.",
       llm_config=llm_config,
   )
   ```

These agents are invoked through a series of API calls and scripts managed by Autogen. The process typically follows these steps:

1. **Initialization**: The agents are initialized with their respective configurations, including the LLM model and system messages.

   ```python
   config_list = [
       {
           "model": gpt_model,
           "api_key": os.getenv("OPENAI_API_KEY"),
           "base_url": os.getenv("AZURE_OPENAI_ENDPOINT"),
           "api_version": os.getenv("OPENAI_API_VERSION"),
           "api_type": "azure",
       }
   ]

   llm_config = {
       "cache_seed": 1,
       "temperature": 0,
       "config_list": config_list,
       "timeout": 120
   }
   ```

2. **Group Chat**: A group chat is created with the User Proxy Agent, Coder Agent, and Product Manager Agent. This chat facilitates communication and collaboration between the agents.

   ```python
   group_chat = autogen.GroupChat(
       agents=[user_proxy, coder, pm], messages=[], max_round=15
   )
   ```

3. **Chat Management**: A Group Chat Manager is used to manage the group chat and ensure that the agents work together effectively.

   ```python
   manager = autogen.GroupChatManager(groupchat=group_chat, llm_config=llm_config)
   ```

4. **User Input**: The User Proxy Agent initiates the chat by sending a message with the game requirements. The other agents then collaborate to generate the game code based on these requirements.

   ```python
   user_proxy.initiate_chat(
       manager,
       message="""
       I would like to create a space invader game in Python! Make sure the game ends when the player gets hit by the enemy red lasers.
       Make sure the game should have a score counter that increases by 1 each time the player hits an enemy with blue lasers.
       The enemies should move from left to right and right to left on the screen and also shoot red lasers at the player randomly.
       Make sure that when the enemy laser (Red lasers) hit the player, the game should display a message saying "Game Over".
       When all the enemies are destroyed, the game should display a message saying "You Win" and display the final score.
       The "Game Over" page should display the final score along with a restart button to restart the game and a quit button to quit the game.
       When the player hits the restart button, the game should restart with the initial number of enemies at the initial level and the score should be reset to 0.
       """,
   )
   ```

By leveraging these AI agents, the project aims to streamline the game development process, making it faster and more efficient.
