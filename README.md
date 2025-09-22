# rate_limit.py
import psycopg2
import time

DB_PARAMS = {
    "dbname": "ratelimit_db",
    "user": "postgres",
    "password": "password",
    "host": "localhost",
    "port": 5432
}

def check_rate_limit(user_id):
    conn = psycopg2.connect(**DB_PARAMS)
    cur = conn.cursor()
    cur.execute("""
        SELECT COUNT(*)
        FROM api_calls
        WHERE user_id = %s
        AND call_time > now() - interval '1 minute'
    """, (user_id,))
    count = cur.fetchone()[0]
    if count >= 5:
        print("Rate limit exceeded for user", user_id)
    else:
        cur.execute("INSERT INTO api_calls(user_id) VALUES(%s)", (user_id,))
        conn.commit()
        print("API call allowed for user", user_id)
    cur.close()
    conn.close()

if __name__ == "__main__":
    for i in range(7):
        check_rate_limit(1)
        time.sleep(5)
-- schema.sql
CREATE TABLE api_calls (
    id SERIAL PRIMARY KEY,
    user_id INT,
    call_time TIMESTAMP DEFAULT now()
);
