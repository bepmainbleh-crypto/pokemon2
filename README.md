# pokemon2
# 🎮 Async Pokémon Battle Simulator

A text-based Pokémon battle simulation program implemented in Python. This project demonstrates advanced **Object-Oriented Programming (OOP)** principles such as *Inheritance* and *Polymorphism*, alongside **Asynchronous Programming** to fetch real-time data from an external API.

---

## ✨ Key Features

- **PokeAPI Integration:** Pokémon names and image URLs are fetched dynamically and asynchronously from [PokeAPI](https://pokeapi.co) using `aiohttp`.
- **Class-Based Pokémon System (Polymorphism):**
  - **Base Pokémon (Generic):** Features randomized base stats and standard combat/feeding behaviors.
  - **🧙‍♂️ Wizard:** Features a passive ability with a 20% chance to cast a shield, completely blocking incoming enemy attacks. Due to a fast metabolism, it can be fed every **10 seconds**.
  - **🥊 Fighter:** Unleashes a powerful `super_power` attack that inflicts randomized bonus damage. However, it takes longer to digest food and can only be fed every **40 seconds**.
- **Time-Based Feeding Mechanic (Cooldown):** Real-time cooldown mechanics for HP recovery powered by Python's built-in `datetime` module. If a user attempts to feed their Pokémon too early, the system precisely calculates the remaining wait time.

---

## 🛠️ Prerequisites & Installation

Ensure you have Python (version 3.8 or higher) installed on your machine.

1. **Clone or Copy the Script**
   Save the core program into a file named `pokemon_game.py`.

2. **Install Dependencies**
   This application relies on `aiohttp` to perform asynchronous HTTP requests. Install it via your terminal or CMD:
   ```bash
   pip install aiohttp
   ```

---

## 🚀 How to Run

Simply execute the script using Python in your terminal:

```bash
python pokemon_game.py
```

---

## 🌐 PokeAPI JSON Schema Overview

The program requests data from the following endpoint using a randomized Pokémon ID (1–1000):
`https://pokeapi.coapi/v2/pokemon/{random_id}`

While the PokeAPI JSON payload is massive, this codebase filters out two primary structures:

1. **Pokémon Name (`get_name`)**
   Retrieved from the first form entry array of the physical Pokémon structure:
   ```json
   {
     "forms": [
       {
         "name": "bulbasaur",
         "url": "https://pokeapi.coapi/v2/pokemon-form/1/"
       }
     ]
   }
   ```
   *Accessed via code using:* `data['forms']['name']`

2. **Pokémon Sprite (`show_img`)**
   Retrieved from the default front-facing sprite object:
   ```json
   {
     "sprites": {
       "front_default": "https://githubusercontent.com"
     }
   }
   ```
   *Accessed via code using:* `data['sprites']['front_default']`

---

## 🤖 Chat Bot Integration Guide (Discord / Telegram)

Since all core methods inside the Pokémon classes are built using asynchronous functions (`async def`), this structure is **fully out-of-the-box compatible** with modern bot frameworks.

### 1. Integration Example for Discord.py
Ensure you have installed the library via `pip install discord.py`.

```python
import discord
from discord.ext import commands
from pokemon_game import Fighter, Wizard # Importing the classes from your script

bot = commands.Bot(command_prefix="!", intents=discord.Intents.all())
player_pokemons = {} # Global dictionary to store player pokemons

@bot.command()
async def claim(ctx, poke_type: str):
    # User types: !claim fighter OR !claim wizard
    trainer = ctx.author.name
    if poke_type.lower() == "fighter":
        player_pokemons[trainer] = Fighter(trainer)
    else:
        player_pokemons[trainer] = Wizard(trainer)
        
    poke = player_pokemons[trainer]
    info_text = await poke.info()
    img_url = await poke.show_img()
    
    # Send a Rich Embed message to Discord
    embed = discord.Embed(title=f"{trainer}'s Pokémon", description=info_text)
    if img_url:
        embed.set_image(url=img_url)
    await ctx.send(embed=embed)

@bot.command()
async def attack(ctx, target: discord.Member):
    # User types: !attack @FriendName
    attacker = ctx.author.name
    defender = target.name
    
    if attacker in player_pokemons and defender in player_pokemons:
        battle_result = await player_pokemons[attacker].attack(player_pokemons[defender])
        await ctx.send(battle_result)
    else:
        await ctx.send("One of you hasn't claimed a Pokémon yet! Type !claim")
```

### 2. Integration Example for Python-Telegram-Bot
Ensure you have installed the library via `pip install python-telegram-bot`.

```python
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes
from pokemon_game import Fighter, Wizard

user_pokemon = {}

async def start_pokemon(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user.username or update.effective_user.first_name
    # Automatically register them with a Fighter class instance
    user_pokemon[user] = Fighter(user)
    
    poke = user_pokemon[user]
    status = await poke.info()
    img = await poke.show_img()
    
    await update.message.reply_text(f"Welcome @{user}!\n\n{status}")
    if img:
        await update.message.reply_photo(photo=img)

async def feed_pokemon(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user.username or update.effective_user.first_name
    if user in user_pokemon:
        response = await user_pokemon[user].feed()
        await update.message.reply_text(response)
    else:
        await update.message.reply_text("Please type /start first to acquire your Pokémon!")

# Run the Application initialization with your Telegram Bot Token
```

---

## 🤝 Contributing
Contributions, bug reports, and feature expansions (such as a level-up progression engine or PvP tournament modes) are highly encouraged. Feel free to open an *Issue* or submit a *Pull Request*!
