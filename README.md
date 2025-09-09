import PySimpleGUI as sg
from secretsharing import PlaintextToHexSecretSharer
from web3 import Web3
import requests
import json
import time

# ====== ดึง token จาก address ======
def get_all_tokens(address):
    url = f"https://api.ethplorer.io/getAddressInfo/{address}?apiKey=freekey"
    resp = requests.get(url)
    data = resp.json()
    tokens = []
    for t in data.get("tokens", []):
        token_info = t['tokenInfo']
        tokens.append({
            "contract": token_info['address'],
            "symbol": token_info.get('symbol', ''),
            "decimals": int(token_info.get('decimals', 18)),
            "balance": int(t['balance'])
        })
    return tokens

# =====## Hi there 👋

<!--
**Wasan1916/Wasan1916** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
