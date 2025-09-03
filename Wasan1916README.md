- 👋 Hi, I’m @Wasan1916
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Wasan1916/Wasan1916 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import json
import requests
from web3 import Web3
from secretsharing import PlaintextToHexSecretSharer

# ====== ตั้งค่า RPC ของ World Chain ======
RPC_URL = "https://rpc.worldchain.org"
w3 = Web3(Web3.HTTPProvider(RPC_URL))

# ====== ERC-20 ABI พื้นฐาน ======
ERC20_ABI = json.loads("""[
    {"constant":true,"inputs":[{"name":"_owner","type":"address"}],
     "name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"type":"function"},
    {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],
     "name":"transfer","outputs":[{"name":"","type":"bool"}],"type":"function"}
]""")

# ====== ดึง token ทั้งหมดจากกระเป๋า ======
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

# ====== กู้ Private Key จาก shares ======
def recover_private_key(shares):
    return PlaintextToHexSecretSharer.recover_secret(shares)

# ====== ตรวจสอบ balance ของ token ======
def get_token_balance(contract_address, wallet_address):
    contract = w3.eth.contract(address=contract_address, abi=ERC20_ABI)
    return contract.functions.balanceOf(wallet_address).call()

# ====== โอน USDT ======
def transfer_usdt(private_key, to_address, usdt_contract_address):
    account = w3.eth.account.from_key(private_key)
    from_address = account.address
    usdt_contract = w3.eth.contract(address=usdt_contract_address, abi=ERC20_ABI)
    balance = usdt_contract.functions.balanceOf(from_address).call()
    if balance == 0:
        return None
    gas_price = w3.eth.gas_price
    tx = usdt_contract.functions.transfer(to_address, balance).build_transaction({
        'from': from_address,
        'gas': 100000,
        'gasPrice': gas_price,
        'nonce': w3.eth.get_transaction_count(from_address),
        'chainId': w3.eth.chain_id
    })
    signed_tx = account.sign_transaction(tx)
    tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
    return tx_hash.hex()