# Stock-portfolio-
from flask import Flask, request, jsonify, render_template
import sqlite3
import requests

app = Flask(__name__)

# Alpha Vantage API Configuration
ALPHA_VANTAGE_API_KEY = 'YOUR_ALPHA_VANTAGE_API_KEY'
BASE_URL = 'https://www.alphavantage.co/query'

# Initialize SQLite database
DATABASE = 'portfolio.db'

def init_db():
    conn = sqlite3.connect(DATABASE)
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS portfolio
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                 symbol TEXT NOT NULL,
                 quantity INTEGER NOT NULL,
                 purchase_price REAL NOT NULL)''')
    conn.commit()
    conn.close()

init_db()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/portfolio', methods=['GET'])
def get_portfolio():
    conn = sqlite3.connect(DATABASE)
    c = conn.cursor()
    c.execute("SELECT * FROM portfolio")
    stocks = c.fetchall()
    conn.close()
    return jsonify(stocks)

@app.route('/api/add_stock', methods=['POST'])
def add_stock():
    data = request.json
    symbol = data['symbol']
    quantity = data['quantity']
    purchase_price = data['purchase_price']

    conn = sqlite3.connect(DATABASE)
    c = conn.cursor()
    c.execute("INSERT INTO portfolio (symbol, quantity, purchase_price) VALUES (?, ?, ?)",
              (symbol, quantity, purchase_price))
    conn.commit()
    conn.close()

    return jsonify({"message": "Stock added successfully"}), 201

@app.route('/api/remove_stock/<int:id>', methods=['DELETE'])
def remove_stock(id):
    conn = sqlite3.connect(DATABASE)
    c = conn.cursor()
    c.execute("DELETE FROM portfolio WHERE id = ?", (id,))
    conn.commit()
    conn.close()

    return jsonify({"message": "Stock removed successfully"}), 200

@app.route('/api/stock_price/<string:symbol>', methods=['GET'])
def get_stock_price(symbol):
    params = {
        'function': 'GLOBAL_QUOTE',
        'symbol': symbol,
        'apikey': ALPHA_VANTAGE_API_KEY
    }
    response = requests.get(BASE_URL, params=params)
    data = response.json()

    if 'Global Quote' in data:
        price = data['Global Quote']['05. price']
        return jsonify({"symbol": symbol, "price": price})
    else:
        return jsonify({"error": "Stock not found"}), 404

if __name__ == '__main__':
    app.run(debug=True)