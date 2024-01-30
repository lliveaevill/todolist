from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
import sqlite3
import uvicorn
from pydantic import BaseModel

app = FastAPI()

# データベースのパス
db_path = "todolist.db"
class Task(BaseModel):
    task: str
# データベース接続のコンテキストマネージャ
def create_connection():
    conn = sqlite3.connect(db_path)
    return conn

# テーブル作成 児島さんが２
def create_table(conn):
     cursor = conn.cursor()
     cursor.execute("""
         CREATE TABLE IF NOT EXISTS tasks (
             id INTEGER PRIMARY KEY AUTOINCREMENT,
             task_text TEXT
         )
     """)
     conn.commit()

# タスクをデータベースに追加
def add_task(task_text):
    conn = create_connection()
    create_table(conn)
    cursor = conn.cursor()
    cursor.execute("INSERT INTO tasks (task_text) VALUES (?)", (task_text,))
    conn.commit()
    conn.close()

# タスクリストを取得
def get_tasks(): #児島さんが
     conn = create_connection()
     create_table(conn)
     cursor = conn.cursor()
     cursor.execute("SELECT task_text FROM tasks")#元々cursor.execute("SELECT * FROM tasks")
     tasks = cursor.fetchall()
     conn.close()
     return tasks

# HTML ファイルを返すエンドポイント
@app.get("/")
async def read_root(request: Request):
    # ここで templates 変数を定義するか、関数外でグローバルに定義する
    templates = Jinja2Templates(directory="templates")
    return templates.TemplateResponse("index.html", {"request": request})

# タスクをデータベースに追加するエンドポイント
@app.post("/add_task")
async def add_task_endpoint(task: Task):
    print(5)
    add_task(task.task)#task
    return {"status": "Task added successfully"}

# タスクリストを取得するエンドポイント
@app.get("/get_tasks")
async def get_tasks_endpoint():
    tasks = get_tasks()
    print(tasks)
    return {"tasks": tasks}

# HTML ファイルを提供するためのディレクトリを設定
app.mount("/static", StaticFiles(directory="static"), name="static")

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
