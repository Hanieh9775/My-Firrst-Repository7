import requests
from flask import Flask, request, render_template_string

app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Currency Converter</title>
</head>
<body>
    <h1>Currency Converter</h1>
    <form method="post">
        <input type="number" step="0.01" name="amount" placeholder="Amount" required>
        <input type="text" name="from_currency" placeholder="From (e.g. USD)" required>
        <input type="text" name="to_currency" placeholder="To (e.g. EUR)" required>
        <button type="submit">Convert</button>
    </form>
    {% if result %}
        <h2>Converted Amount: {{ result }}</h2>
    {% endif %}
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def home():
    result = None
    if request.method == "POST":
        amount = request.form.get("amount")
        from_currency = request.form.get("from_currency").upper()
        to_currency = request.form.get("to_currency").upper()
        try:
            response = requests.get(f"https://api.exchangerate.host/convert?from={from_currency}&to={to_currency}&amount={amount}", timeout=5)
            data = response.json()
            if "result" in data:
                result = f"{data['result']} {to_currency}"
            else:
                result = "Conversion failed."
        except Exception:
            result = "Error fetching conversion data."
    return render_template_string(HTML_TEMPLATE, result=result)

if __name__ == "__main__":
    app.run(debug=True)
