"""
FastAPI Dynamic CRUD Demo - Single File

Description:
- In-memory database (dict), dynamic CRUD for any "table" or JSON.
- Automatic console demonstration of CRUD operations.
- Interactive Swagger UI for testing endpoints.
- Data is NOT persistent (lost on server restart).

Requirements:
- Python 3.10+
- FastAPI
- Uvicorn
Install:
    pip install fastapi uvicorn

Run:
    python main.py
    Server will be at http://127.0.0.1:8000
    Swagger UI: http://127.0.0.1:8000/docs
"""

from fastapi import FastAPI, HTTPException
from typing import Dict, Any

app = FastAPI(title="Dynamic CRUD API")

# In-memory DB
db: Dict[str, list[Dict[str, Any]]] = {}

# -----------------------
# CREATE
# -----------------------
@app.post("/{table_name}/")
def create_item(table_name: str, item: Dict[str, Any]):
    db.setdefault(table_name, [])
    if 'id' in item:
        for existing in db[table_name]:
            if existing.get('id') == item['id']:
                raise HTTPException(status_code=400, detail="Item with this ID already exists")
    db[table_name].append(item)
    return item

# -----------------------
# READ ALL
# -----------------------
@app.get("/{table_name}/")
def get_items(table_name: str):
    return db.get(table_name, [])

# -----------------------
# READ ONE
# -----------------------
@app.get("/{table_name}/{item_id}")
def get_item(table_name: str, item_id: int):
    for item in db.get(table_name, []):
        if item.get('id') == item_id:
            return item
    raise HTTPException(status_code=404, detail="Item not found")

# -----------------------
# UPDATE
# -----------------------
@app.put("/{table_name}/{item_id}")
def update_item(table_name: str, item_id: int, updated_item: Dict[str, Any]):
    table = db.get(table_name, [])
    for i, item in enumerate(table):
        if item.get('id') == item_id:
            table[i] = updated_item
            return updated_item
    raise HTTPException(status_code=404, detail="Item not found")

# -----------------------
# DELETE
# -----------------------
@app.delete("/{table_name}/{item_id}")
def delete_item(table_name: str, item_id: int):
    table = db.get(table_name, [])
    for i, item in enumerate(table):
        if item.get('id') == item_id:
            del table[i]
            return {"message": "Item deleted"}
    raise HTTPException(status_code=404, detail="Item not found")

# -----------------------
# Console CRUD Demo
# -----------------------
if __name__ == "__main__":
    # Prepare DB for testing
    table = "my_table"
    db.setdefault(table, [])

    # CREATE
    print("\n--- CREATE ---")
    item1 = {"id": 1, "name": "Example 1", "description": "Description 1"}
    item2 = {"id": 2, "name": "Example 2", "description": "Description 2"}
    db[table].append(item1)
    db[table].append(item2)
    print("Items after create:", db[table])

    # READ ALL
    print("\n--- READ ALL ---")
    print(db[table])

    # READ ONE
    print("\n--- READ ONE (id=1) ---")
    item = next((i for i in db[table] if i["id"] == 1), None)
    print(item)

    # UPDATE
    print("\n--- UPDATE (id=1) ---")
    updated_item = {"id": 1, "name": "Example 1 - Updated", "description": "New Description"}
    for i, it in enumerate(db[table]):
        if it["id"] == 1:
            db[table][i] = updated_item
    print(db[table])

    # DELETE
    print("\n--- DELETE (id=2) ---")
    db[table] = [i for i in db[table] if i["id"] != 2]
    print(db[table])

    # Run FastAPI server (optional)
    import uvicorn
    uvicorn.run("main:app", host="127.0.0.1", port=8000, reload=True)
