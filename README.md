# PROJECT
web_scanner_project/
├── scanner.py
├── scan_report.json
├── requirements.txt
├── README.md
├── templates/
│   └── index.html
├── static/
│   └── style.css (optional)
└── app.py

1. scanner.py – Core scanning logic
   import requests
from bs4 import BeautifulSoup
import json
from urllib.parse import urljoin

def find_forms(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")
    return soup.find_all("form")

def submit_form(form, url, value):
    action = form.get("action")
    method = form.get("method", "get").lower()
    inputs = form.find_all("input")
    data = {}

    for input in inputs:
        name = input.get("name")
        type = input.get("type", "text")
        if name:
            data[name] = value if type == "text" else input.get("value", "")

    full_url = urljoin(url, action)
    if method == "post":
        return requests.post(full_url, data=data)
    else:
        return requests.get(full_url, params=data)

def scan(url):
    report = []
    forms = find_forms(url)
    for form in forms:
        response = submit_form(form, url, "<script>alert(1)</script>")
        if "<script>alert(1)</script>" in response.text:
            report.append({"vulnerability": "XSS", "url": url})
    
    with open("scan_report.json", "w") as f:
        json.dump(report, f, indent=4)

    return report
2. app.py – Flask Web Interface
   from flask import Flask, render_template, request
from scanner import scan

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    report = None
    if request.method == "POST":
        url = request.form.get("url")
        report = scan(url)
    return render_template("index.html", report=report)

if __name__ == "__main__":
    app.run(debug=True)
3. templates/index.html – Simple User Form
<!DOCTYPE html>
<html>
<head>
    <title>Web Scanner</title>
</head>
<body>
    <h1>Web Application Vulnerability Scanner</h1>
    <form method="POST">
        <input type="text" name="url" placeholder="Enter URL" required />
        <button type="submit">Scan</button>
    </form>
    {% if report %}
        <h2>Scan Results:</h2>
        <ul>
        {% for item in report %}
            <li>{{ item.vulnerability }} found at {{ item.url }}</li>
        {% endfor %}
        </ul>
    {% endif %}
</body>
</html>
 4. requirements.txt – Dependencies
 Flask
requests
beautifulsoup4
