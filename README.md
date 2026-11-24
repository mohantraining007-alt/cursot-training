# Demo Server Monitoring

Local demo that shows resource usage for this Linux host in the browser.

## Setup

Run these steps from the project directory:

1. Create a virtual environment: `python3 -m venv .venv`
2. Activate it: `source .venv/bin/activate`
3. Install dependencies: `pip install -r requirements.txt`
4. Start the app: `python app.py`
5. Open `http://127.0.0.1:5254`

The app listens on `127.0.0.1:5254` only. The home page shows the product name and a short description. Sign-in and live metrics come in later steps.
