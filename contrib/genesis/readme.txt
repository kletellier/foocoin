Usage: genesis.exe [options]

Options:
  -h, --help show this help message and exit
  -t TIME, --time=TIME  the (unix) time when the genesisblock is created
  -z TIMESTAMP, --timestamp=TIMESTAMP
     the pszTimestamp found in the coinbase of the genesisblock
  -n NONCE, --nonce=NONCE
     the first value of the nonce that will be incremented
     when searching the genesis hash
  -a ALGORITHM, --algorithm=ALGORITHM
     the PoW algorithm: [SHA256]
  -p PUBKEY, --pubkey=PUBKEY
     the pubkey found in the output script
  -v VALUE, --value=VALUE
     the value in coins for the output, full value (exp. in bitcoin 5000000000 - To get other coins value: Block Value * 100000000)

