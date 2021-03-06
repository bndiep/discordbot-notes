# discordbot-notes

## Resources
- [Building a Discord Bot with Python and Repl.it](https://www.codementor.io/@garethdwyer/building-a-discord-bot-with-python-and-repl-it-miblcwejz)
- [How to Make a Discord Bot in Python](https://realpython.com/how-to-make-a-discord-bot-python/)

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

## Bot Life
- Need to keep bot alive when Repl is running
- Do so by creating a server to run it in a separate thread using the Flask framework
```
from flask import Flask
from threading import Thread
  
app = Flask('')

@app.route('/')
def home():
  return "I'm alive"

def run():
  app.run(host='0.0.0.0',port=8080)
  
def keep_alive():
  t = Thread(target=run)
  t.start
```
- Starts a web server that returns "I'm alive" if anyone visits it and provides metho to start in a new thread
- Add import of server to top of `main.py` like so: `from keep_alive import keep_alive`
- Call `keep_alive()` just before the secret token declaration
- When you run again, you should see a new pane that shows the web output from the server
- Flask will be listening for requests
- May have to login to Repl to start the bot again (after long inactivity)

## API Implementation
- Currently using [OpenWeatherMap API](https://openweathermap.org/current)
- Be sure to sign up and get a free API key to use as a token
- Make a 'GET' request:
```
response = requests.get(f'https://api.openweathermap.org/data/2.5/weather?zip=92108,us&appid={weather_api}')
```
- Select the correct endpoint:
```
weather_description = response.json()['weather'][0]['description']
weather_temp = response.json()['main']['temp']
```

## Make Bot Respond to Messages
- This will be a client.event
- Check to see if the user's message is equal to the user's input 
- Protect again recursion to ensure that the bot sends the message to a normal user and not to itself--want to compare the author's message to the bot user (i.e. client.user)
```
if message.author == client.user:
  return
```

## Handling Errors
- Raise exceptions by calling `on_error`
- `on_error` takes the event as the first argument (which will be  `on_message` in this case)
- It also excepts `*args` and `**kwargs` which are flexible, positional, and keyword arguments
- Handle the exception with a unique message by using `.write()` to format the string to the file  `err.log`
- `on_message` takes one arg `message` so the first argument in the list should be the user's input
```
@client.event
async def on_error(event, *args, **kwargs):
  with open('err.log', 'a') as f:
  if event == 'on_message':
    f.write(f'Unhandled message: {args[0]}\n')
  else:
    raise
```

## Bot Commands
- Command is an object that wraps a function that is invooked by a text command
- Starts with a  `command_prefix ` and defined by the Bot
`bot = commands.Bot(command_prefix='!'`
- Use `client.command` and pass the command name as the argument (e.g. `@bot.command(name='99')
- The command the user inputs will be '!{command_name}' where command_name is the unique identifier
- The callback function accepts on parameter which is the context surrounding the invoked Command
- Example:
```
bot = commands.Bot(command_prefix='!')

@bot.command(name='marco')
async def marco(context):
  response = 'polo!'
  await context.send(response)
```
- Add a help message to give a clear description of what the command does when the user types of the help command:
```
bot.command(name="marco", help="Responds with 'Polo' when the user calls for Marco)
```
*** NOTE: This functionality only exists for the Bot subclass NOT for Client superclass ***
*** IMPORTANT: By using command decorators like the one above AND having an `on_message` event, you need the bot to process commands! You need this otherwise the commands WILL NOT TRIGGER. Save yourself the headache and change `on_message` to as follows:
```
@bot.event
async def on_message(message):
  await bot.process_commands(message)
```
