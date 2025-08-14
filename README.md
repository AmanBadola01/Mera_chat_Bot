# Mera_chat_Bot

A simple chatbot that lets users order food from “Pandeyji Eatery.”  
Backend is a FastAPI webhook, orders are saved in MySQL, and there’s a small static website with the menu.

What you can do
- Add items to an order, remove items, and complete the order
- Store order items and create an order-tracking row
- Get total price of an order (via a MySQL function)
- Check order status by order ID
- View a basic landing page with images

Project layout
- home.html — Landing page
- styles.css — Styles for the page
- main.py — FastAPI app (webhook for Dialogflow)
- db_helper.py — MySQL calls (stored procedure, function, queries)
- generic_helper.py — Small helpers (format order text, get session id)
- images/ — Banner and menu images

Requirements
- Python 3.10+
- MySQL 8+
- pip
- Optional: Dialogflow agent (for full chatbot flow)

Quick start

1) Clone and create a virtual env
- git clone  Mera_chat_Bot
- cd Mera_chat_Bot
- python -m venv .venv
- activate it:
  - macOS/Linux: source .venv/bin/activate
  - Windows: .venv\Scripts\activate

2) Install packages
- pip install fastapi uvicorn mysql-connector-python

3) Set DB credentials
- Open db_helper.py and set host, user, password, database.

4) Prepare the database (minimal)
- CREATE DATABASE pandeyji_eatery; USE pandeyji_eatery;
- Create tables orders, order_items, order_tracking (any schema works that supports items with price and a tracking status).
- Create these DB objects expected by code:
  - Stored procedure: insert_order_item(food_item, quantity, order_id)
  - Function: get_total_order_price(order_id)
  Simple example ideas:
  - orders(order_id INT PRIMARY KEY)
  - order_items(order_id INT, food_item VARCHAR(100), quantity INT, price DECIMAL(10,2))
  - order_tracking(order_id INT, status VARCHAR(50))

5) Run the server
- uvicorn main:app --reload --port 8000

6) Test quickly
- Send a POST to http://localhost:8000/ with a Dialogflow-style payload.
- Or connect this URL as a webhook in Dialogflow.

Dialogflow notes
- Intent names expected:
  - order.add - context: ongoing-order
  - order.remove - context: ongoing-order
  - order.complete - context: ongoing-order
  - track.order - context: ongoing-tracking
- Parameters:
  - food-item (array of item names)
  - number (array of quantities)
  - order_id (number for tracking)
- Ensure outputContexts include a path with /sessions/.../contexts/.

How it works (short)
- The app keeps in-progress orders in memory per session_id.
- On complete, it:
  - Picks next order_id (MAX(order_id)+1),
  - Calls insert_order_item for each item,
  - Adds an “in progress” row to order_tracking,
  - Gets total using get_total_order_price,
  - Returns order_id and total.

Static site
- Open home.html in a browser to view the menu and banner.
- You can serve it with: python -m http.server

Tips and known issues
- styles.css has broken braces; consider replacing it with a clean stylesheet.
- Using MAX(order_id)+1 can clash under load. Prefer an AUTO_INCREMENT orders table.
- Don’t hardcode DB passwords; use environment variables in real setups.
- In-memory orders reset when the server restarts.
