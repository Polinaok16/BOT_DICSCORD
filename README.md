# BOT_DICSCORD

Бот включает в себя событие on_ready, при которым выводится сообщение в консоль, что бот на старте, и создается таблица users, которая заполняется id членов сервера.

```
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
