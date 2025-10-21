import sqlite3
import typer

app = typer.Typer()

# ------------------------------
# Función para conectar a la base de datos
# ------------------------------
def get_db_connection():
    conn = sqlite3.connect("todo.db")
    conn.execute("""
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            task TEXT NOT NULL
        )
    """)
    return conn


# ------------------------------
# Agregar una nueva tarea
# ------------------------------
@app.command()
def add(task: str):
    """Agrega una nueva tarea a la lista."""
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO tasks (task) VALUES (?)", (task,))
    conn.commit()
    conn.close()
    typer.echo(f"✅ Tarea agregada: {task}")


# ------------------------------
# Listar todas las tareas
# ------------------------------
@app.command()
def list():
    """Muestra todas las tareas guardadas."""
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT id, task FROM tasks")
    rows = cursor.fetchall()
    conn.close()

    if not rows:
        typer.echo("📭 No hay tareas registradas.")
        return

    typer.echo("📋 Lista de tareas:")
    for row in rows:
        typer.echo(f"{row[0]}. {row[1]}")


# ------------------------------
# Eliminar una tarea
# ------------------------------
@app.command()
def delete(task_id: int):
    """Elimina una tarea por su ID."""
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM tasks WHERE id = ?", (task_id,))
    if not cursor.fetchone():
        typer.echo(f"⚠️ No se encontró tarea con ID {task_id}.")
        conn.close()
        return

    cursor.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
    conn.commit()
    conn.close()
    typer.echo(f"🗑️ Tarea eliminada correctamente (ID: {task_id})")


# ------------------------------
# Ejecutar la aplicación
# ------------------------------
if __name__ == "_main_":
    app()
