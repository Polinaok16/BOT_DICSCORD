# BOT_DICSCORD

Бот включает в себя событие on_ready, при которым выводится сообщение в консоль, что бот на старте, и создается таблица users, которая заполняется id членов сервера.

```
@bot.event
async def on_ready():
    cursor.execute("""CREATE TABLE IF NOT EXISTS users (
        id INT,
        count INT
    )""")
    connection.commit()
    
    for guild in bot.guilds:
        for member in guild.members:
            if cursor.execute(f"SELECT id FROM users WHERE id = {member.id}").fetchone() is None:
                cursor.execute(f"INSERT INTO users VALUES ({member.id},0)")
            else:
                pass
    connection.commit()
    print(f'{bot.user.name} на старте')
```
Далее идет событие на присоединение нового члена (on_member_join) в канал и на удаление члена (on_member_remove) из канала. При данных событиях просто выводятся сообщения ботом. 

```
@bot.event
async def on_member_join(member):
    await member.send('Привет! Набери !info для просмотра команд')
    if cursor.execute(f"SELECT id FROM users WHERE id = {member.id}").fetchone() is None:
        cursor.execute(f"INSERT INTO users VALUES ({member.id},0)")
        connection.commit()
    else:
        pass
    for ch in bot.get_guild(member.guild.id).channels:
        if ch.name == 'основной':
            await bot.get_channel(ch.id).send(f'{member},круто, что ты присоединился!')
    
@bot.event
async def on_member_remove(member):
    for ch in bot.get_guild(member.guild.id).channels:
        if ch.name == 'основной':
            await bot.get_channel(ch.id).send(f'{member}, жаль, будет тебя не хватать!')
    
```

Далее создаются команды для бота. Бот имеет семь команд. Команда !информация информируется пользователя о том, какие команды имеет бот. 

```
@bot.command()
async def информация(ctx, arg = None):
    author = ctx.message.author
    if arg == None:
        await ctx.send(f'{author.mention} Введите:\n!информация общая\n!info команды')
    elif arg == 'общая':
        await ctx.send(f'{author.mention} Я Bot2 и я отвечаю за порядок и веселье в чате')
    elif arg == 'команды':
        await ctx.send(f'{author.mention} !test - Бот онлайн?\n !статус - позитив/негатив\n !рандом - рандомное число\n !баланс - баланс пользователя\n !награда - начисление средств\n !списывание - списывание средств')
    else:
        await ctx.send(f'{author.mention} Такой команды нет!')
```
 Команад !test проверяет в сети ли бот и выводит картинку котика. 

```
@bot.command()
async def test(ctx):
     await ctx.send('Я тут', file = discord.File('images.jpeg'))
```

Команда !статус проверяет сколько негативных и позитивных слов написал пользователь боту и ведет их подсчет, возвращая пользователю итог. 

```
@bot.command()
async def статус(ctx):
    if neg == 0:
        await ctx.send('вы не использовали негативные слова')
    else:
        await ctx.send('вы использовали негативные слова в количестве')
        await ctx.send(neg)
    if pos == 0:
        await ctx.send('вы не использовали  позитивные слова')
    else:
        await ctx.send('вы использовали позитивные слова в количестве')
        await ctx.send(pos)
```      

Команда !баланс позволяет посмотреть баланс свой или другого члена (колонка count в таблице users) и возвращает значение из таблицы. Если автор команды запросил свой баланс, не указав другого члена, то получаем его по ctx.author.id, а если после названия команды указан другой член, то возвращаем значение баланса по member.id.

```     
@bot.command()
async def баланс(ctx, member: discord.Member = None):
    if member is None:
        await ctx.send(embed = discord.Embed(
            description = f"""Баланс пользователя **{ctx.author}** составляет **{cursor.execute("SELECT count FROM users WHERE id = {}".format(ctx.author.id)).fetchone()[0]}:leaves:**"""
        ))
    else:
         await ctx.send(embed = discord.Embed(
            description = f"""Баланс пользователя **{member}** составляет **{cursor.execute("SELECT count FROM users WHERE id = {}".format(member.id)).fetchone()[0]} :leaves:**"""
        ))
```     
