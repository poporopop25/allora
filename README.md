# Allora Worker [PROPHET MODEL]
1. System Requirements:
  - Operating System : Ubuntu 22.04
  - CPU: Minimum of 1/2 core.
  - Memory: 2 to 4 GB.
  - Storage: SSD or NVMe with at least 5GB of space.
2. Running on VPS:
  - Buy here: https://www.kqzyfj.com/click-101201397-13484397
  - Choose VPS1, European union, Storage type - SSD, Ubuntu 22.04, The rest is default
  - Enter your password. fill up the customer details and choose your payment method
  - Wait for confirmation of your order and your login details [separate email]
3. Get testnet token:
  - Go to: https://faucet.testnet-1.testnet.allora.network/

# Docker Compose Installation
```bash
# Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

# Install Docker-Compose
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version

# Docker Permission to user
sudo groupadd docker
sudo usermod -aG docker $USER
```
Clean old docker
```bash
docker compose down -v
docker container prune
cd $HOME && rm -rf allora-huggingface-walkthrough
```
# Deployment - you must read and execute it properly
# Step 1
```bash
git clone https://github.com/allora-network/allora-huggingface-walkthrough
cd allora-huggingface-walkthrough
```
# Step 2
```bash
cp config.example.json config.json
nano config.json
```
# Step 3 
- Edit config.json copy & replace
```bash
{
    "wallet": {
        "addressKeyName": "YOUR_WALLET_NAME",
        "addressRestoreMnemonic": "SEED_PHRASE",
        "alloraHomeDir": "/root/.allorad",
        "gas": "1000000",
        "gasAdjustment": 1.0,
        "nodeRpc": "https://allora-rpc.testnet-1.testnet.allora.network/",
        "maxRetries": 1,
        "delay": 1,
        "submitTx": false
    },
    "worker": [
        {
            "topicId": 1,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 1,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "ETH"
            }
        },
        {
            "topicId": 2,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 3,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "ETH"
            }
        },
        {
            "topicId": 3,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 5,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "BTC"
            }
        },
        {
            "topicId": 4,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 2,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "BTC"
            }
        },
        {
            "topicId": 5,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 4,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "SOL"
            }
        },
        {
            "topicId": 6,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 5,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "SOL"
            }
        },
        {
            "topicId": 7,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 2,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "ETH"
            }
        },
        {
            "topicId": 8,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 3,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "BNB"
            }
        },
        {
            "topicId": 9,
            "inferenceEntrypointName": "api-worker-reputer",
            "loopSeconds": 5,
            "parameters": {
                "InferenceEndpoint": "http://inference:8000/inference/{Token}",
                "Token": "ARB"
            }
        }
        
    ]
}
```
- <b> Replace <i>YOUR_WALLET_NAME & SEED_PHRASE</i></b>
- To save the config.json file `Ctrl+X Y ENTER`
- Edit RPC if its not working: https://sentries-rpc.testnet-1.testnet.allora.network/
# Step 4
```bash
chmod +x init.config
./init.config
```
# Step 5
- Edit app.py copy & replace
- Register on Coingecko & create demo account https://www.coingecko.com/en/developers/dashboard & Create Demo API KEY
- Replace API with your `COINGECKO API` , then save `Ctrl+X Y ENTER`.
```bash
nano app.py
```
```bash
from flask import Flask, Response
import requests
import json
import pandas as pd
from prophet import Prophet

# create our Flask app
app = Flask(__name__)

def get_coingecko_url(token):
    base_url = "https://api.coingecko.com/api/v3/coins/"
    token_map = {
        'ETH': 'ethereum',
        'SOL': 'solana',
        'BTC': 'bitcoin',
        'BNB': 'binancecoin',
        'ARB': 'arbitrum'
    }
    
    token = token.upper()
    if token in token_map:
        url = f"{base_url}{token_map[token]}/market_chart?vs_currency=usd&days=30&interval=daily"
        return url
    else:
        raise ValueError("Unsupported token")

def handle_outliers(df):
    # Removing outliers using IQR (Interquartile Range) method
    Q1 = df['y'].quantile(0.25)
    Q3 = df['y'].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    df = df[(df['y'] >= lower_bound) & (df['y'] <= upper_bound)]
    return df

# define our endpoint
@app.route("/inference/<string:token>")
def get_inference(token):
    """Generate inference for given token."""
    try:
        # Get the data from Coingecko
        url = get_coingecko_url(token)
    except ValueError as e:
        return Response(json.dumps({"error": str(e)}), status=400, mimetype='application/json')

    headers = {
        "accept": "application/json",
        "x-cg-demo-api-key": "CG-XXXXXXXXXXXXXXXXXXXXXXXXX"  # Replace with your API key
    }

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        data = response.json()
        df = pd.DataFrame(data["prices"])
        df.columns = ["ds", "y"]
        df["ds"] = pd.to_datetime(df["ds"], unit='ms')
        df = df[:-1]  # Removing today's price
        df = handle_outliers(df)  # Handling outliers
        print(df.tail(5))
    else:
        return Response(json.dumps({"Failed to retrieve data from the API": str(response.text)}),
                        status=response.status_code,
                        mimetype='application/json')

    # Fit the model using Prophet with custom parameters
    model = Prophet(
        seasonality_mode='multiplicative', 
        changepoint_prior_scale=0.05,
        seasonality_prior_scale=10.0,
        holidays_prior_scale=10.0
    )

    # Adding custom weekly seasonality
    model.add_seasonality(name='weekly', period=7, fourier_order=3)
    model.fit(df)

    # Make a forecast for the next 7 days
    future = model.make_future_dataframe(periods=7)
    forecast = model.predict(future)

    # Get the forecasted values for the next 7 days
    forecasted_values = forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail(7).to_dict(orient='records')
    print(forecasted_values)  # Print the forecasted values
 return Response(json.dumps(forecasted_values), status=200, mimetype='application/json')

# run our Flask app
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8002, debug=True)
```
# Step 6
```bash
nano requirements.txt
```
Copy & replace, then save `Ctrl+X Y ENTER`
```bash
flask[async]
gunicorn[gthread]
transformers[torch]
pandas
torch==2.0.1 
python-dotenv
requests==2.31.0
plotly
prophet
```
# Step 7
Run the docker
```bash
docker compose up --build -d
```
# Register here to track your points: https://app.allora.network?ref=eyJyZWZlcnJlcl9pZCI6IjA1ZThhODY3LWY0MGMtNGIwYi1hNGZjLTkwNGI5NjUyNzcyYyJ9
# Check your wallet here: http://worker-tx.nodium.xyz/
![image](https://github.com/user-attachments/assets/6e9ce7fd-fdf5-40d2-98f9-d20eb8486fce)
# Other Command
Check worker logs
```bash
docker logs -f worker
```
Docker down
```bash
docker compose down
```
Docker up
```bash
docker compose up -d
```
Thank to 0xtnpxsgt for the guide!

Join my telegram for your question - https://t.me/airdropPH2024room
