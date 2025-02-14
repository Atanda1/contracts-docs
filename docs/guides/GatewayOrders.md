# Gateway Orders

To create orders, users have to 


```
// Off ramp on paycrest

const { SimulationProvider } = zimulatoor;
const { Contract, formatUnits, parseUnits, BigNumber, ZeroAddress } = ethers;
const { address: usdtAddress, abi: usdtAbi } = bonadocs.contracts.USDT;
const { address: gatewayAddress, abi: gatewayAbi } = bonadocs.contracts.Gateway;

// Initating variables
const chainId = 42161
const provider = new SimulationProvider(chainId)
const senderAddress = '0xF977814e90dA44bFA03b6295A0616a897441aceC';

// Initiating signers
const senderSigner = await provider.getImpersonatedSigner(senderAddress);

// Initiating contracts
const gateway = new Contract(gatewayAddress, gatewayAbi, senderSigner)
const usdtAsset = new Contract(
  usdtAddress,
  usdtAbi,
  senderSigner,
);

// Encrypt arbitrary data with a public key
async function publicKeyEncrypt(data, publicKeyPEM) {
  // First, we need to convert PEM to a format Web Crypto API can use
  function pemToArrayBuffer(pem) {
    const b64 = pem.replace(/-----BEGIN PUBLIC KEY-----/, '')
      .replace(/-----END PUBLIC KEY-----/, '')
      .replace(/\s/g, '');
    const binary = atob(b64);
    const arr = new Uint8Array(binary.length);
    for (let i = 0; i < binary.length; i++) {
      arr[i] = binary.charCodeAt(i);
    }
    return arr.buffer;
  }

  // Convert PEM to ArrayBuffer
  const publicKeyBuffer = pemToArrayBuffer(publicKeyPEM);

  // Import the public key
  const publicKey = await self.crypto.subtle.importKey(
    'spki',
    publicKeyBuffer,
    {
      name: 'RSA-OAEP',
      hash: 'SHA-256',
    },
    true,
    ['encrypt']
  );

  // Encrypt the data
  const encoder = new TextEncoder();
  const dataBuffer = encoder.encode(JSON.stringify(data));
  const encrypted = await self.crypto.subtle.encrypt(
    {
      name: 'RSA-OAEP'
    },
    publicKey,
    dataBuffer
  );

  // Convert the encrypted data to base64 for easy transmission
  return btoa(String.fromCharCode.apply(null, new Uint8Array(encrypted)));
}

// Fetch public key of aggregator
const fetchAggregatorPublicKey = async () => {
  try {
    const response = await fetch("https://api.paycrest.io/v1/pubkey");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Error fetching aggregator public key:", error);
    throw error;
  }
};

// Encrypt recipient details
const recipient = {
  accountIdentifier: "2002948489",
  accountName: "Chibuotu  Amadi",
  institution: "KUDANGPC",
  providerId: "etoMCRIY",
  memo: "N/A",
};

const publicKey = await fetchAggregatorPublicKey();
const messageHash = await publicKeyEncrypt(recipient, publicKey.data);

console.log(publicKey, messageHash);

// approve on USDT
const usdtAmount = parseUnits('100', 6);

const approveTx =
  await usdtAsset.approve(gatewayAddress, usdtAmount);

const approveRct = await approveTx.wait();

console.log(approveRct.logs)

// create order through the gateway contract

const refundAddress = "0xFf7dAD16C6Cd58FD0De22ddABbcBF35f888Fc9B2"

try {
  const createOrderTx = await gateway.createOrder(
    usdtAddress,
    parseUnits('10', 6),
    1674,
    ZeroAddress,
    0,
    refundAddress,
    messageHash
  )

  const createOrderRct = await createOrderTx.wait();

  const orderLog = createOrderRct.logs.find((l) => l.fragment?.name === "OrderCreated")
  
  console.log("emittedCreatedOrder", {
    refundAddress: orderLog.args[0],
    token: orderLog.args[1],
    amount: orderLog.args[2],
    fee: orderLog.args[3],
    orderId: orderLog.args[4],
    rate: orderLog.args[5],
    messageHash: orderLog.args[6]
  })
} catch (error) {
  parseEthersError(error, sablier)
  console.error(error)
}
```