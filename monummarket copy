import discord
from discord.ext import commands
import asyncio
import datetime
from fuzzywuzzy import fuzz
import requests
import discord.ext
from discord.ext.commands import Paginator
import json
import os

from discord_slash import SlashCommand
from discord_slash import SlashContext

disTOKEN = ''

bot = commands.Bot(command_prefix='/')
bot.remove_command('help')
slash = SlashCommand(bot, sync_commands=True)
DATABASE = "data.json"


def load_data():
    try:
        with open('data.json', 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        return {}

items = {}
sellers = {}
selling_items = []
history = {}
items = load_data()
CHANNEL_ID = 1076468080370262137

server_address = "server.playmonumenta.com"
server_port = 25565

@bot.event
async def on_ready():
    global items, history
    with open(DATABASE, "r") as f:
        data = json.load(f)
        items = data.get("items", {})
        history = data.get("history", {})

def format_user(user: discord.Member) -> str:
    return f"<@{user.id}>"

async def set_status():
    await bot.wait_until_ready()
    while not bot.is_closed():
        servers = list(bot.guilds)
        members = set()
        for server in servers:
            members.update(server.members)
        await asyncio.sleep(10)
        await bot.change_presence(activity=discord.Game(name="/help"))
        await asyncio.sleep(10)
        await bot.change_presence(activity=discord.Game(name="by veeshfrade#0305"))
        await asyncio.sleep(10)
        
        try:
            response = requests.get(f"https://api.minetools.eu/ping/{server_address}/{server_port}")
            if response.status_code == 200:
                player_count = response.json()['players']['online']
                activity = discord.Activity(type=discord.ActivityType.watching, name=f"{player_count} Monumenta players")
                await bot.change_presence(activity=activity)
            else:
                await bot.change_presence(activity=discord.Game(name="Server Offline"))
        except:
            await bot.change_presence(activity=discord.Game(name="Error"))
        
        await asyncio.sleep(10)

bot.loop.create_task(set_status())



def format_user(user: discord.Member) -> str:
    return f"<@{user.id}>"

def convert_coins_to_supercoins(coins):
    supercoins = coins // 64
    coins = coins % 64
    return supercoins, coins


@bot.command(name='offer')
async def cmd_offer(ctx, *item_info):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    if len(item_info) < 2:
        embed = discord.Embed(title="Error", description="Please specify the `item name` and `price`.", color=discord.Color.red())
        await ctx.send(embed=embed)
        return
    item_name = item_info[0]
    item_price = item_info[1]
    if len(item_info) > 2:
        item_info = " ".join(item_info[2:])
        if len(item_info) > 100:
            item_info = item_info[:100]
        else:
            item_info = item_info
    else:
        item_info = ""
    with open(DATABASE, "r") as f:
        items = json.load(f)
    sellers = items.setdefault(item_name, [])
    for seller_tuple in sellers:
        if len(seller_tuple) != 3:
            print(f"Invalid seller tuple in {item_name}: {seller_tuple}")
            continue
        seller, price, info = seller_tuple
        if seller == ctx.author.name:
            embed = discord.Embed(title="Error", description=f"You have already added `{item_name}` for sale.", color=discord.Color.red())
            await ctx.send(embed=embed)
            return
    sellers.append((ctx.author.name, item_price, item_info))
    items[item_name] = sellers
    with open(DATABASE, "w") as f:
        json.dump(items, f, indent=4)
    embed = discord.Embed(title="Selling", description=f"Item `{item_name}` added for sale! ({item_info})", color=discord.Color.blue())
    await ctx.send(embed=embed)


@bot.command(name='get')
async def cmd_get(ctx, item_name):
    print(f"[{ctx.message.created_at}] Message from {ctx.author.name}: {ctx.message.content}")
    with open(DATABASE, "r") as f:
        items = json.load(f)
    def find_similar_items(item_name):
        similar_items = []
        for name in items:
            ratio = fuzz.ratio(item_name.lower(), name.lower())
            if ratio > 55:
                similar_items.append(name)
        return similar_items
    if item_name in items:
        sellers_list = []
        for seller, price, item_info in items[item_name]:
            info = f' ({item_info})' if len(item_info) > 0 else ''
            sellers_list.append(f"{seller} is selling `{item_name}` for `{price}`{info}")
        embed = discord.Embed(title=f"People who are selling `{item_name}`", description="\n".join(sellers_list), color=discord.Color.blue())
        await ctx.send(embed=embed)
    else:
        similar_items = find_similar_items(item_name)
        if similar_items:
            embed = discord.Embed(title="Item not found", description=f"Did you mean : `{', '.join(similar_items)}`?", color=discord.Color.red())
            await ctx.send(embed=embed)
        else:
            embed = discord.Embed(title="Error",description="Item not found", color=discord.Color.red())
            await ctx.send(embed=embed)
with open(DATABASE, "w") as f:
        json.dump(items, f, indent=4)


@bot.command(name='offerlist')
async def cmd_offerlist(ctx):
    print(f"[{ctx.message.created_at}] Message from {ctx.author.name}: {ctx.message.content}")
    with open(DATABASE, "r") as f:
        items = json.load(f)
    sellers = []
    for item_name in items:
        for seller, price, info in items[item_name]:
            if seller not in sellers:
                sellers.append(f"{seller} is selling `{item_name}` for `{price}` ({info})")
    if sellers:
        seller_count = len(sellers)
        embed = discord.Embed(title=f"People who are selling items( Results: `{seller_count}` )", description="\n".join(sellers), color=discord.Color.blue())
        await ctx.send(embed=embed)
    else:
        embed = discord.Embed(title="No sellers found", color=discord.Color.red())
        await ctx.send(embed=embed)
with open(DATABASE, "w") as f:
        json.dump(items, f, indent=4)

@bot.command(name='myoffers')
async def cmd_myoffers(ctx):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    with open(DATABASE, "r") as f:
        items = json.load(f)
    seller = ctx.author
    selling_items = []
    for item_name, sellers in items.items():
        for index, seller_tuple in enumerate(sellers):
            if len(seller_tuple) == 2:
                seller_name, item_price = seller_tuple
                if seller_name == seller.name:
                    selling_items.append(f"{item_name} for {item_price}")
            elif len(seller_tuple) == 3:
                seller_name = seller_tuple[0]
                item_price = seller_tuple[1]
                if seller_name == seller.name:
                    selling_items.append(f"`{item_name}` for `{item_price}` {seller_tuple[2]}")
            else:
                print(f"Invalid seller tuple at index {index} in {item_name}: {seller_tuple}")
    if selling_items:
        selling_items_formatted = "\n".join(selling_items)
        embed = discord.Embed(title=f"Items {seller.name} is selling", description=selling_items_formatted, color=discord.Color.blue())
        await ctx.send(embed=embed)
    else:
        embed = discord.Embed(title="No items being sold", description="You are not selling any items.", color=discord.Color.red())
        embed.set_author(name=ctx.author.name, icon_url=str(ctx.author.avatar_url))
        await ctx.send(embed=embed)

@bot.command(name='offercancel')
async def cmd_offercancel(ctx):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    with open(DATABASE, "r") as f:
        items = json.load(f)
    try:
        item_name = ctx.message.content.split()[1]
    except IndexError:
        embed = discord.Embed(title="Item not found", description="Please provide the item name after the `~offercancel` command.", color=discord.Color.red())
        await ctx.send(embed=embed)
        return

    if item_name in items:
        removed = False
        for i, (seller, price, *details) in enumerate(items[item_name]):
            if seller == ctx.author.name or str(seller) == str(ctx.author.id):
                del items[item_name][i]
                if not items[item_name]:
                    del items[item_name]
                embed = discord.Embed(title="Item removing", description=f"Item `{item_name}` has been removed.", color=discord.Color.blue())
                await ctx.send(embed=embed)
                removed = True
                break
        if not removed:
            embed = discord.Embed(title="Item not found", description=f"Item `{item_name}` not found for sale by you.", color=discord.Color.red())
            await ctx.send(embed=embed)
        else:
            # Add removed item to history
            user_id = str(ctx.author.id)
            item_details = (item_name, price) + tuple(details)
            history.setdefault(user_id, []).append(item_details)
        with open (DATABASE, "w") as f:
            json.dump(items, f, indent=4)
    else:
        embed = discord.Embed(title="Item not found", description=f"Item `{item_name}` not found in the list of items for sale.", color=discord.Color.red())
        await ctx.send(embed=embed)

    # Add removed item to history
        user_id = str(ctx.author.id)
    item_details = (item_name, price) + tuple(details)
    history.setdefault(user_id, []).append(item_details)

@bot.command(name='ofhistory')
async def cmd_ofhistory(ctx):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    user_id = str(ctx.author.id)
    user_history = history.get(user_id, [])
    removed_items = set()  
    description = None
    for item in user_history:
        name, price, *details = item
        if name not in removed_items:  
            removed_items.add(name)
            details_str = "\n".join(details) if details else "-"
            if description is None:
                description = f"Item: {name}\nPrice: {price}\nDetails: {details_str}\n"
            else:
                description += f"\nItem: {name}\nPrice: {price}\nDetails: {details_str}\n"
    if description:
        title = "Removed Items"
        color = discord.Color.blue()
        embed = discord.Embed(title=title, description=description, color=color)
        await ctx.send(embed=embed)
    else:
        title = "Removed Items"
        description = "You haven't removed any items yet."
        color = discord.Color.red()
        embed = discord.Embed(title=title, description=description, color=color)
        await ctx.send(embed=embed)

@bot.command(name='clearofhistory')
async def cmd_clearofhistory(ctx):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    user_id = str(ctx.author.id)
    if user_id in history:
        del history[user_id]
        embed = discord.Embed(title="History cleared", description="Your history of removed items has been cleared.", color=discord.Color.blue())
        await ctx.send(embed=embed)
    else:
        embed = discord.Embed(title="History not found", description="You don't have any history of removed items.", color=discord.Color.red())
        await ctx.send(embed=embed)


@bot.command(name='mcalc')
async def mcalc(ctx, quantity: int, price: float):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    if quantity is None or price is None:
        return await ctx.send("You must provide the quantity and price of the item(s) you wish to purchase.")
    total_cost = quantity * price
    supercoins, coins = convert_coins_to_supercoins(total_cost)
    embed = discord.Embed(title="Money Calculation Results", color=discord.Color.blue())
    embed.add_field(name="Quantity", value=f"{quantity} items")
    embed.add_field(name="Price", value=f"{price} C$$ for each")
    embed.add_field(name="Total Cost", value=f"{total_cost} C$$")
    if supercoins > 0:
        embed.add_field(name="Cost in H$$", value=f"{supercoins}")
    if coins > 0:
        embed.add_field(name="Cost in C$$", value=f"{coins}")
    await ctx.send(embed=embed)


@slash.slash(name="help", description="The main command for the correct use of the bot.")
async def cmd_help(ctx: SlashContext):
    print(f"[{ctx.created_at}] Slash command '{ctx.name}' invoked by {ctx.author.name}")
    embed = discord.Embed(title="Help with bot", color=discord.Color.blue())
    embed.set_thumbnail(url=bot.user.avatar_url)
    embed.add_field(name="Buying (getting)", value="`/get <itemname>` : This command allows you to buy an item specified as a command parameter. If no item is found, the bot will try to find the most similar items and offer them.\n*Example :* */get sword*", inline=False)
    embed.add_field(name="Selling", value="`/offer <itemname> <price>` `<and more info if you want to keep it>` : This command allows the user to sell his item. The item and the price must be specified as parameters of the command.\n*Example :* */offer sword 1hcs in barrel*\nor\n*/offer sword 1har*", inline=False)
    embed.add_field(name="Global item list ", value="`/offerlist` : This command displays all customers and all their items", inline=False)
    embed.add_field(name="Yours items for selling", value="`/myoffers` : This command shows all the items you sell", inline=False)
    embed.add_field(name="Help", value="`/help` : This command invokes the menu that you see", inline=False)
    embed.add_field(name="Report a bugs", value="`/bugreport <text>` : This command allows you to report bugs you find\n*Example : /bugreport the bot does not respond to the /get command*", inline=False)
    embed.add_field(name="Item cancel(removing)", value="`/offercancel <your itemname>` : This command allows you to remove your items from sale\n*Example : /offercancel sword*", inline=False)
    embed.add_field(name="Your idea", value="`/idea <your idea>` : This command allows you to send your ideas directly to me\n*Example : /idea new command /exchange*", inline=False)
    embed.add_field(name="link to invite the bot to the server (if you want)", value="https://discord.com/api/oauth2/authorize?client_id=1068915740226355240&permissions=8&scope=bot%20applications.commands", inline=False)
    embed.add_field(name="Money calculator", value="`/mcalc <quanity> <price> ` : This command allows you to calculate the price of items that you buy on the server market\n*Example : /mcalc 48 35*", inline=False)
    embed.add_field(name="History of canceled offers", value="`/ofhistory` : This command allows you to view all your deleted items", inline=False)
    embed.add_field(name="Clear history of canceled offers", value="`/clearofhistory` : This command will clear your history of deleted items", inline=False)
    embed.add_field(name="Other information about the bot", value="Bot created in the python programming language.\nThe name of the bot was invented by a neural network\nWhen searching for items(when buying certain items) you can write the name of the item incorrectly, but the bot will analyze all the items and display the most similar.", inline=False)
    await ctx.send(embed=embed)


@bot.command(name='idea')
async def cmd_idea(ctx, *, idea):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    developer_channel = bot.get_channel(1076515049671241869)
    await developer_channel.send(f'Idea from {ctx.author.name}: {idea}')
    embed = discord.Embed(
        title="Thank you for your idea!",
        description=f"{idea}",
        color=discord.Color.blue()
    )
    embed.set_footer(text=f"Idea submitted by {ctx.author.name}")
    await ctx.send(embed=embed)


@bot.command(name='bugreport')
async def cmd_bugreport(ctx, *bug_description):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    bug_description = ' '.join(bug_description)
    developer_channel = bot.get_channel(1076468080370262137)
    report_embed = discord.Embed(title="Bug Report", description=bug_description, color=discord.Color.blue())
    report_embed.set_author(name=ctx.author.name, icon_url=ctx.author.avatar_url)
    await developer_channel.send(embed=report_embed)
    confirm_embed = discord.Embed(title="Thank you for your bug report!", color=discord.Color.blue())
    await ctx.send(embed=confirm_embed)

@bot.event
async def on_command_error(ctx, error):
    print(f"[{datetime.datetime.now()}] Message from {ctx.author.name}: {ctx.message.content}")
    if isinstance(error, commands.CommandNotFound):
        embed = discord.Embed(
            title="Error",
            description="The command you entered - does not exist.",
            color=discord.Color.red()
        )
        embed.set_footer(text="Write `/help` to help with commands and the bot.")
        await ctx.send(embed=embed)

@bot.event
async def on_ready():
    print('--------------------------------')
    print('         BOT IS ONLINE!'     )
    print('Bot username:', bot.user.name,bot.user.discriminator)
    print('--------------------------------')

bot.run(disTOKEN)