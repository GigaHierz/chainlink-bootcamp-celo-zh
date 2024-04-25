# Chainlink开发者训练营--CELO 特邀嘉宾专场

本资源库是[Chainlink Developer Bootcamp 2024](https://lu.ma/ChainlinkBootcamp2024?utm_source=0czoelvgwhbx) 中关于 Celo 演示的一部分。

点击此处查看幻灯片：

## 安装 Celo-Composer 

开始使用 Celo Composer 的最简单方法是使用 [`@celo/celo-composer`](https://github.com/celo-org/celo-composer)。这个 CLI 工具可让您快速开始在 Celo 上为多个框架构建 dApp，包括 React（使用 react-celo 或 rainbowkit-celo）。开始使用，只需运行以下命令并按步骤操作即可：

```bash
npx @celo/celo-composer@latest create
```

### 安装所有依赖程序

run

```bash
npm i
```

or 

```bash
yarn
```

## 获取令牌余额

在 `index.tsx` 文件中，你会发现一些模板代码。我们将在其中添加我们的代码。

首先，我们要读取 CELO 代币的余额。





1. 为此，我们需要获取 CELO 在 Alfajores 上的地址。您可以在 [Celo docs](https://docs.celo.org/token-addresses) 中找到地址。或者在[celoscan](https://celoscan.io/tokens)中查看顶级ERC20代币。

```typescript
    const CELOTokenAddress = "0xF194afDf50B03e69Bd7D057c1Aa9e10c9954E4C9"; // CELO 测试网

```

2. 从 viem 导入 ERC20 abi

```typescript
 //  Import an eRC20 abi from viem
    import { erc20Abi } from "viem";
```

3. we will use [web3.js](https://web3js.org/) to read from the contracts. so, we need to create an instance, initialize it with the RPC for Celo Alfajores.

Install web.js

```bash
yarn add web3
```

Import Web3.js


```typescript
 
```

Create the web3 instance. You will need an RPC. You can find the Celo RPCs in the [Celo docs](https://docs.celo.org/network)

```typescript
    import Web3 from "web3";
```


4. Create a Contract instance of the celo contract to be able to interact with it
```typescript
    const celoContract = new web3.eth.Contract(erc20Abi, CELOTokenAddress)
```

5. Call the contract function to read the users current CELO balance.

```typescript
    // save the balance value in the state
    const [celoBalance, setCeloBalance] = useState("");

    const getBalance = async () => {
        celoContract.methods.balanceOf(userAddress as `0x${string}`).call()
            .then(balance => {
                // Balance is returned in Wei, convert it to Ether (or token's equivalent)
                const tokenBalance = web3.utils.fromWei(balance, 'ether');
                setCeloBalance((tokenBalance))
                console.log(`The balance is: ${tokenBalance}`);
            })
            .catch(error => {
                console.error(error);
            });

        return;
    };
```

6. Call the function in a react hook above. The hook already exists and we add out code

```typescript
    useEffect(() => {
        if (isConnected && address) {
            setUserAddress(address);

            // add this code, as we only want to load the user balance, once we are connected and the component is mounted.
            if (isMounted) {
                getBalance();
            }
        }
        // isMounted needs to be added here as well
    }, [address, isConnected, isMounted]);
```

7. Add some code to the HTML to show the users current amount of CELO tokens

```typescript
    return (
        <div className="flex flex-col justify-center items-center">
            <div className="h1">
                There you go... a canvas for your next Celo project!
            </div>
            {isConnected ? (
                <div className="h2 text-center">
                    Your address: {userAddress}

                    {/* add your values here, we will shorten it to two decimal values */}
                    <p> CELO Balance {Number(celoBalance).toFixed(2)}</p>
                
                </div>

            ) : (
                <div>No Wallet Connected</div>
            )}
        </div>
    );
```


Your code should now look likes this: 

```typescript
// ERC20 abi 

export default function Home() {
    //  Import an eRC20 abi from viem
    import { erc20Abi } from "viem";

    // CELO token address on Alfajore Celo Testnet
    const CELOTokenAddress = "0xF194afDf50B03e69Bd7D057c1Aa9e10c9954E4C9"; 

    const web3 = new Web3("https://alfajores-forno.celo-testnet.org")
    const celoContract = new web3.eth.Contract(erc20Abi, CELOTokenAddress)

    const [userAddress, setUserAddress] = useState("");
    const [isMounted, setIsMounted] = useState(false);
    // create a state for the celoBalance
    const [celoBalance, setCeloBalance ]  = useState("");

      useEffect(() => {
        setIsMounted(true);
    }, []);

    useEffect(() => {
        if (isConnected && address) {
            setUserAddress(address);

            if (isMounted) {
                getBalance();
            }
        }
    }, [address, isConnected, isMounted]);

    if (!isMounted) {
        return null;
    }

    const getBalance = async () => {
          celoContract.methods.balanceOf(userAddress as `0x${string}`).call()
            .then(balance => {
                // Balance is returned in Wei, convert it to Ether (or token's equivalent)
                const tokenBalance = web3.utils.fromWei(balance, 'ether');
                setCeloBalance((tokenBalance))
                console.log(`The balance is: ${tokenBalance}`);
            })
            .catch(error => {
                console.error(error);
            });

        return;
    };

        return (
        <div className="flex flex-col justify-center items-center">
            <div className="h1">
                There you go... a canvas for your next Celo project!
            </div>
            {isConnected ? (
                <div className="h2 text-center">
                    Your address: {userAddress}
                    {/* add your values here, we will shorten it to two decimal values */}
                    <p> CELO Balance {Number(celoBalance).toFixed(2)}</p>
                </div>

            ) : (
                <div>No Wallet Connected</div>
            )}
        </div>
    );
}
```

## Get USD value of CELO tokens

For this tutorial we will follow the guide from the [Chainlink docs](https://docs.chain.link/data-feeds/using-data-feeds#reading-data-feeds-offchain) for reading data prcie feeds offchain


1. Find the address of the CELO/USD pricefeed in the [Chainlink docs ](https://docs.chain.link/data-feeds/price-feeds/addresses?network=celo&page=1#overview)

```typescript
    // pricefeed address for CELO/USD on Alfajores
    const celoToUsd = "0x022F9dCC73C5Fb43F2b4eF2EF9ad3eDD1D853946";
```

2. Add the aggregatorV3InterfaceABI from the tutorial 
```typescript
    // pricefeed address for CELO/USD on Alfajores
    const aggregatorV3InterfaceABI = [
    {
        inputs: [],
        name: "decimals",
        outputs: [{ internalType: "uint8", name: "", type: "uint8" }],
        stateMutability: "view",
        type: "function",
    },
    {
        inputs: [],
        name: "description",
        outputs: [{ internalType: "string", name: "", type: "string" }],
        stateMutability: "view",
        type: "function",
    },
    {
        inputs: [{ internalType: "uint80", name: "_roundId", type: "uint80" }],
        name: "getRoundData",
        outputs: [
        { internalType: "uint80", name: "roundId", type: "uint80" },
        { internalType: "int256", name: "answer", type: "int256" },
        { internalType: "uint256", name: "startedAt", type: "uint256" },
        { internalType: "uint256", name: "updatedAt", type: "uint256" },
        { internalType: "uint80", name: "answeredInRound", type: "uint80" },
        ],
        stateMutability: "view",
        type: "function",
    },
    {
        inputs: [],
        name: "latestRoundData",
        outputs: [
        { internalType: "uint80", name: "roundId", type: "uint80" },
        { internalType: "int256", name: "answer", type: "int256" },
        { internalType: "uint256", name: "startedAt", type: "uint256" },
        { internalType: "uint256", name: "updatedAt", type: "uint256" },
        { internalType: "uint80", name: "answeredInRound", type: "uint80" },
        ],
        stateMutability: "view",
        type: "function",
    },
    {
        inputs: [],
        name: "version",
        outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
        stateMutability: "view",
        type: "function",
    },
    ]
```


3. Create the contract instance for the priceFeed contract

```typescript
    // contract instance of the price feed contract
    const priceFeed = new web3.eth.Contract(aggregatorV3InterfaceABI, celoToUsd)
```

4. Call the latestRound data to get the latest price data. This is the response data that you will get, so you will want to read the second value
```typescript
    0: roundID,
    1: answer, // this is the value we want
    3: timeStamp,
    2: startedAt,
    4: answeredInRound
```

First again let's add the state to store our celoValue

```typescript
   // store the celoValue in the state
    const [celoValue, setCeloValue] = useState("");
```

Then the function to call the price feed data


```typescript
    const getUSDValue = async () => {
    priceFeed.methods
        .latestRoundData()
        .call()
        .then((roundData: any) => {
            // get the value from position one of the response object. The value will come back as bigInt so we will have to format it. 
            const balance = (formatUnits(roundData[1], 8))
            setCeloValue(balance)
            // Do something with roundData
            console.log("Latest Round Data", roundData)
        })
    };

```

Call the function in our hook, when the dApp is connected with a wallet and the component is mounted

```typescript
    useEffect(() => {
        if (isConnected && address) {
            setUserAddress(address);
            if (isMounted) {
                getBalance();
                getUSDValue()
            }
        }
    }, [address, isConnected, isMounted]);
```

Add some code to display the celoValue to the user

```typescript
    return (
        <div className="flex flex-col justify-center items-center">
            <div className="h1">
                There you go... a canvas for your next Celo project!
            </div>
            {isConnected ? (
                <div className="h2 text-center">
                    Your address: {userAddress}
                     {/* add your values here, we will shorten it to two decimal values */}
                    <p> CELO Balance {Number(celoBalance).toFixed(2)}</p>
                    <p> USD value of CELO {(Number(celoBalance) * Number(celoValue)).toFixed(2)}</p>
                </div>
            ) : (
                <div>No Wallet Connected</div>
            )}
        </div>
    );
```

Now your code should look like this


```typescript
export default function Home() {
    const celoToUsd = "0x022F9dCC73C5Fb43F2b4eF2EF9ad3eDD1D853946"; // Price Feed Contract Address. You can find it here: https://docs.chain.link/data-feeds/price-feeds/addresses?network=celo&page=1#overview

    const [celoValue, setCeloValue ]  = useState("");

   const getUSDValue = async () => {
        priceFeed.methods
            .latestRoundData()
            .call()
            .then((roundData: any) => {
                const balance = (formatUnits(roundData[1], 8))
                setCeloValue(balance)
                // Do something with roundData
                console.log("Latest Round Data", roundData)
            })
    };
}
```

## Final code

Let's add some code to showcase the values to the user. Your whole code should look like this now. And we are done. Congratulations. You now know how to implement price feed data into your dApp. 

```typescript
import { useEffect, useState } from "react";
import { useAccount } from "wagmi";
//  Import an eRC20 abi from viem
import { erc20Abi, formatUnits } from "viem";
import Web3 from "web3";

export default function Home() {
    const CELOTokenAddress = "0xF194afDf50B03e69Bd7D057c1Aa9e10c9954E4C9"; // CELO Testnet
    // pricefeed address for CELO/USD on Alfajores
    const celoToUsd = "0x022F9dCC73C5Fb43F2b4eF2EF9ad3eDD1D853946";
    // create a Web3 instance, initialize it with the RPC for Celo Alfajores
    // pricefeed address for CELO/USD on Alfajores
    const aggregatorV3InterfaceABI = [
        {
            inputs: [],
            name: "decimals",
            outputs: [{ internalType: "uint8", name: "", type: "uint8" }],
            stateMutability: "view",
            type: "function",
        },
        {
            inputs: [],
            name: "description",
            outputs: [{ internalType: "string", name: "", type: "string" }],
            stateMutability: "view",
            type: "function",
        },
        {
            inputs: [{ internalType: "uint80", name: "_roundId", type: "uint80" }],
            name: "getRoundData",
            outputs: [
                { internalType: "uint80", name: "roundId", type: "uint80" },
                { internalType: "int256", name: "answer", type: "int256" },
                { internalType: "uint256", name: "startedAt", type: "uint256" },
                { internalType: "uint256", name: "updatedAt", type: "uint256" },
                { internalType: "uint80", name: "answeredInRound", type: "uint80" },
            ],
            stateMutability: "view",
            type: "function",
        },
        {
            inputs: [],
            name: "latestRoundData",
            outputs: [
                { internalType: "uint80", name: "roundId", type: "uint80" },
                { internalType: "int256", name: "answer", type: "int256" },
                { internalType: "uint256", name: "startedAt", type: "uint256" },
                { internalType: "uint256", name: "updatedAt", type: "uint256" },
                { internalType: "uint80", name: "answeredInRound", type: "uint80" },
            ],
            stateMutability: "view",
            type: "function",
        },
        {
            inputs: [],
            name: "version",
            outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
            stateMutability: "view",
            type: "function",
        },
    ]
    const web3 = new Web3("https://alfajores-forno.celo-testnet.org")
    const celoContract = new web3.eth.Contract(erc20Abi, CELOTokenAddress)
    // contract instance of the price feed contract
    const priceFeed = new web3.eth.Contract(aggregatorV3InterfaceABI, celoToUsd)


    const [userAddress, setUserAddress] = useState("");
    const [isMounted, setIsMounted] = useState(false);
    // save the balance value in the state
    const [celoBalance, setCeloBalance] = useState("");
    // store the celoValue in the state
    const [celoValue, setCeloValue] = useState("");

    const { address, isConnected } = useAccount();

    useEffect(() => {
        setIsMounted(true);
    }, []);

    useEffect(() => {
        if (isConnected && address) {
            setUserAddress(address);
            if (isMounted) {
                getBalance();
                getUSDValue()
            }
        }
    }, [address, isConnected, isMounted]);

    if (!isMounted) {
        return null;
    }

    const getBalance = async () => {
        celoContract.methods.balanceOf(userAddress as `0x${string}`).call()
            .then(balance => {
                // Balance is returned in Wei, convert it to Ether (or token's equivalent)
                const tokenBalance = web3.utils.fromWei(balance, 'ether');
                setCeloBalance((tokenBalance))
                console.log(`The balance is: ${tokenBalance}`);
            })
            .catch(error => {
                console.error(error);
            });

        return;
    };

    const getUSDValue = async () => {
        priceFeed.methods
            .latestRoundData()
            .call()
            .then((roundData: any) => {
                // get the value from position one of the response object. The value will come back as bigInt so we will have to format it. 
                const balance = (formatUnits(roundData[1], 8))
                setCeloValue(balance)
                // Do something with roundData
                console.log("Latest Round Data", roundData)
            })
    };

    return (
        <div className="flex flex-col justify-center items-center">
            <div className="h1">
                There you go... a canvas for your next Celo project!
            </div>
            {isConnected ? (
                <div className="h2 text-center">
                    Your address: {userAddress}
                     {/* add your values here, we will shorten it to two decimal values */}
                    <p> CELO Balance {Number(celoBalance).toFixed(2)}</p>
                    <p> USD value of CELO {(Number(celoBalance) * Number(celoValue)).toFixed(2)}</p>
                </div>
            ) : (
                <div>No Wallet Connected</div>
            )}
        </div>
    );
}
```
