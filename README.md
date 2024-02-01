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
