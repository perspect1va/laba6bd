import pyodbc
import random
from datetime import datetime, timedelta

SERVER = 'localhost'
DATABASE = 'User_Actions'

conn = pyodbc.connect(f'DRIVER={{ODBC Driver 17 for SQL Server}};SERVER={SERVER};DATABASE={DATABASE};Trusted_Connection=yes')
cursor = conn.cursor()

for shard_id in range(1, 13):
    cursor.execute(f"""
        IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'shard_{shard_id}')
        BEGIN
            EXEC('CREATE SCHEMA shard_{shard_id}')
        END
    """)
    
    cursor.execute(f"""
        IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'User_Logs_Shard_{shard_id}')
        BEGIN
            CREATE TABLE shard_{shard_id}.User_Logs_Shard_{shard_id} (
                id INT IDENTITY(1,1) NOT NULL,
                username TEXT NOT NULL,
                user_action TEXT NOT NULL,
                action_date DATE NOT NULL,
                action_time TIME NOT NULL,
                action_result TEXT NOT NULL,
                shard_id INT NOT NULL,
                PRIMARY KEY (id, action_date)
            )
        END
    """)
    
    cursor.execute(f"TRUNCATE TABLE shard_{shard_id}.User_Logs_Shard_{shard_id}")
    
    actions = ['LOGIN', 'LOGOUT', 'UPDATE', 'DELETE', 'VIEW', 'CREATE', 'EXPORT', 'IMPORT']
    results = ['SUCCESS', 'FAILED', 'PENDING', 'TIMEOUT', 'ACCESS_DENIED']
    
    start_date = datetime(2025, 1, 1)
    
    for i in range(10000):
        username = f'user_{random.randint(1, 100000):05d}'
        user_action = random.choice(actions)
        days_offset = random.randint(0, 364)
        action_date = start_date + timedelta(days=days_offset)
        random_seconds = random.randint(0, 86399)
        action_time = (datetime.min + timedelta(seconds=random_seconds)).time()
        action_result = random.choice(results)
        
        cursor.execute(f"""
            INSERT INTO shard_{shard_id}.User_Logs_Shard_{shard_id} 
            (username, user_action, action_date, action_time, action_result, shard_id)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (username, user_action, action_date, action_time, action_result, shard_id))
    
    conn.commit()
    print(f'Shard {shard_id} filled with 10000 records')

print('\nStatistics:')
for shard_id in range(1, 13):
    cursor.execute(f"SELECT COUNT(*) FROM shard_{shard_id}.User_Logs_Shard_{shard_id}")
    count = cursor.fetchone()[0]
    print(f'Shard {shard_id}: {count} records')

cursor.close()
conn.close()
