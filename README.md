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

Команда !награда работает похожим образом, но требует помимо ввода ника другого члена, ввода суммы,которую необходимо начислить на баланс пользователю. Если автор не введет, то увидит сообщение с просьбой ввести сумму. Данное значение будет обнволено в колонке count таблицы users.

```   
@bot.command()
async def награда(ctx, member: discord.Member = None, amount: int = None):
    if member is None:
        await ctx.send(f"**{ctx.author}**,укажите пользователя, которому вы хотите выдать денег на счет")
    else:
        if amount is None:
            await ctx.send(f"**{ctx.author}**,укажите сумму, которую вы хотите выдать на счет")
        elif amount < 1:
            await ctx.send(f"**{ctx.author}**,укажите сумму больше 1")
        else:
            cursor.execute("UPDATE users SET count = count + {} WHERE id = {}".format(amount,member.id))
            connection.commit()
            await ctx.message.add_reaction('✔️')
```
Команда !списывание работает так же, требуя ввода ника другого члена и ввода суммы, но для списания с баланса пользователя. Данное значение также обновляется в колонке count таблицы users.

```   
@bot.command()
async def списывание(ctx, member: discord.Member = None, amount = None):
    if member is None:
        await ctx.send(f"**{ctx.author}**,укажите пользователя, которому вы хотите выдать денег на счет")
    else:
        if amount is None:
            await ctx.send(f"**{ctx.author}**,укажите сумму, которую вы хотите выдать на счет")
        elif amount == 'all':
            cursor.execute("UPDATE users SET count = count + {} WHERE id = {}".format(0,member.id))
            connection.commit()
            await ctx.message.add_reaction('✔️')
        elif int(amount) < 1:
            await ctx.send(f"**{ctx.author}**,укажите сумму больше 1")
        else:
            cursor.execute("UPDATE users SET count = count - {} WHERE id = {}".format(int(amount),member.id))
            connection.commit()
            await ctx.message.add_reaction('✔️')
```
Также для дальнейшей обработки настроения речи, с которой общается пользователь бота, потребовалось использование сторонных API.
```   
def request_sentiment(message):
    data = {'x': [message]}
    res = requests.post('https://7015.deeppavlov.ai/model', json=data).json()
    santiment = res[0][0]
    return santiment
```
Дла подсчета того, сколько раз пользователь получил ответ negative и positive были созданы переменные neg = 0 и 
pos = 0. Также было прописано событие on_message, которое во-первых, выводит тип натсроения, с которым общается пользователь (negative,positive,neutral), далее ведет подсчет количества. При введение негативного слова, удаляет его и выводит предупреждение. При использовании ряда ругательств банит пользователя по прифине нецензурной лексики. Также реагирует на вопрос "как дела", пока, привет, и выводит рандомную шутку при запросе при вводе пользователем "расскажи шутку".

```
@bot.event
async def on_message(message):
    global neg, pos
 
    if message.author == bot.user:
        return

    setiment = request_sentiment(message.content)
    await message.channel.send(setiment)
    if setiment == 'negative':
        neg += 1
        await message.channel.send(f'{message.author.mention}, ужас, так нельзя!')
        await message.delete()
    elif setiment == 'positive':
        pos += 1
        await message.channel.send(f'{message.author.mention}, приятно слышать!')
    
    for i in message.content.split(' '):
        if i.lower().translate(str.maketrans('','',string.punctuation)) in ['балбес', 'дерьмо']:
            await message.channel.send(f'{message.author.mention}, это ужасное ругательство!')
            await message.channel.send(f'{message.author.mention}, БАН!')
            await message.author.ban(reason = 'Нецензурная лексика')

    if 'как дела' in message.content.lower():
        await message.channel.send('Нормально')
    elif message.content.lower() == "Привет" or message.content.lower() == "Здраствуйте": 
        await message.channel.send(f'Привет {message.author.mention}') 
        return
    elif message.content.lower() == "Пока": 
        await message.channel.send(f'Пока {message.author.mention}') 
    elif message.content.lower() == "расскажи шутку": 
        jokes = ["Старые мосты могут еще пригодиться. Лучше сжечь старые грабли",
                     "Если план А не сработал, не сдавайся — у тебя есть ещё 32 буквы, чтобы попробовать.",
                     "Это я-то нерешительный? Сомневаюсь…"] 
        await message.channel.send(random.choice(jokes))
```
