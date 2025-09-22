
# CLientサイド

Deploy SafeWallet with Passkey
https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API

## パスキーモジュール付きのSafeWalletを作成
```
const RP_NAME = 'Safe Smart Account'
const USER_DISPLAY_NAME = 'User display name'
const USER_NAME = 'User name'
// https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API
const passkeyCredential = await navigator.credentials.create({
  publicKey: {
    pubKeyCredParams: [
      {
        alg: -7,
        type: 'public-key'
      }
    ],
    challenge: crypto.getRandomValues(new Uint8Array(32)),
    rp: {
      name: RP_NAME
    },
    user: {
      displayName: USER_DISPLAY_NAME,
      id: crypto.getRandomValues(new Uint8Array(32)),
      name: USER_NAME
    },
    timeout: 60_000,
    attestation: 'none'
  }
})

const passkeySigner = await Safe.createPasskeySigner(passkeyCredential)
```


## 2. txを実行(UserOperationを投げる)

```
// aa.ts
import { Safe4337Pack } from '@safe-global/relay-kit'

// 例: Sepolia + Pimlico
const RPC_URL = 'https://rpc.ankr.com/eth_sepolia'
const BUNDLER_URL = `https://api.pimlico.io/v2/11155111/rpc?apikey=YOUR_PIMLICO_KEY`

export async function sendWithAA(passkeySigner, tx) {
  // tx は { to, data, value } の配列にして渡す
  const pack = await Safe4337Pack.init({
    provider: RPC_URL,
    signer: passkeySigner,
    bundlerUrl: BUNDLER_URL,
    options: {
      owners: [],
      threshold: 1 // <----- マルチシグのしきい値. 1はパスキー1つでOK
    },
    paymasterOptions: { isSponsored: true, paymasterUrl: ..., paymasterAddress: ... }
  })

  const op = await pack.createTransaction({ transactions: [tx] })
  const signed = await pack.signSafeOperation(op)// <------- Passkey で署名
  const uoHash = await pack.executeTransaction({ executable: signed })
  return uoHash
}
```