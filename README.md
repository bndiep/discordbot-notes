# discordbot-notes

## Resources
- [Building a Discord Bot with Python and Repl.it](https://www.codementor.io/@garethdwyer/building-a-discord-bot-with-python-and-repl-it-miblcwejz)

## Setting up Discord Bot in Repl.it
- Python wrapper for Discord bot API exists on GitHub
- `import discord` at the top of `main.py` and then 'run'

## Setting up authorization
- Don't share your secret bot token!
- Use environment variables so only you can have access to the token while also giving public access to your code
- Add a special `.env` file
- NOTE: Repl.it no longer allows .env files so use the Environment Variable in the sidebar
- Set key to `DISCORD_BOT_SECRET` and the value should be the secret token
- Follow onscreen instructions to import the environment variable

## Create a Bot that Echos in Reverse
```
import discord
import os

my_secret = os.environ['DISCORD_BOT_SECRET']

client = discord.Client()

@client.event
async def on_ready():
  print("I'm in")
  print(client.user)

@client.event
async def on_message(message):
  if message.author != client:
    await message.channel.send(message.content[::-1])

client.run(my_secret)
```
- Import os to access the bot's secret token
- Create a Discord client (an Py object to send commands to Discord server)
- The event is a Python decorator that takes the code below it and changes it
- Asynchronously run the code when a certain event happens
- `on_ready` event is when the server joins the server successfully
- When it is successfull, on the server side (Repl.it), print `I'm in` and the bot's user id
- Only want to respond to messages not from us
- Send a new message to the same channel where the message was received but in reverse
- - `::-1` is to reverse a string or list in Python
- NOTE: Make sure the non-client messages are unique, otherwise the bot will reverse EVERYTHING (even interfering with other bot messages)
