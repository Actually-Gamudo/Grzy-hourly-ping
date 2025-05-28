import discord
from discord.ext import tasks, commands
import logging

logging.basicConfig(level=logging.INFO)

TOKEN = "YOUR_BOT_TOKEN_HERE"
CHANNEL_ID = 123456789012345678  # Replace with your channel ID
USER_ID = 689447471595782318     # Replace with the user's ID
DEFAULT_MESSAGE = "Hourly ping for <@{user_id}>!"  # Default ping message

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

custom_message = DEFAULT_MESSAGE
pinging_active = False

@bot.event
async def on_ready():
    print(f"✅ Logged in as {bot.user}")
    if pinging_active:
        hourly_ping.start()

@tasks.loop(hours=1)
async def hourly_ping():
    try:
        channel = bot.get_channel(CHANNEL_ID)
        if channel:
            msg = custom_message.format(user_id=USER_ID)
            await channel.send(msg)
        else:
            logging.warning("⚠️ Channel not found.")
    except Exception as e:
        logging.error(f"❌ Error during hourly ping: {e}")

@bot.command(name="startping")
@commands.has_permissions(administrator=True)
async def start_ping(ctx):
    global pinging_active
    if not hourly_ping.is_running():
        hourly_ping.start()
        pinging_active = True
        await ctx.send("✅ Hourly ping started.")
    else:
        await ctx.send("⚠️ Hourly ping is already running.")

@bot.command(name="stopping")
@commands.has_permissions(administrator=True)
async def stop_ping(ctx):
    global pinging_active
    if hourly_ping.is_running():
        hourly_ping.stop()
        pinging_active = False
        await ctx.send("🛑 Hourly ping stopped.")
    else:
        await ctx.send("⚠️ Hourly ping is not currently running.")

@bot.command(name="setmessage")
@commands.has_permissions(administrator=True)
async def set_message(ctx, *, message: str):
    global custom_message
    custom_message = message
    await ctx.send(f"✅ Custom message set to:\n`{message}`")

@bot.command(name="pingstatus")
async def ping_status(ctx):
    status = "🟢 Active" if hourly_ping.is_running() else "🔴 Inactive"
    await ctx.send(f"**Hourly Ping Status:** {status}")

@start_ping.error
@stop_ping.error
@set_message.error
async def permission_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send("❌ You need administrator permissions to use this command.")
    else:
        await ctx.send("❌ An error occurred.")
        logging.error(error)

bot.run(TOKEN)
