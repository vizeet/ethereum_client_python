# ethereum_client_python
Ethereum Client written in Python

Initialize datadirectory with genesis block
geth --datadir data1 init ./genesis.json 

geth --datadir="data1" -verbosity 6 --port 30301 --rpc --rpccorsdomain "http://localhost:8545" --rpcapi "eth,web3,net" --networkid 123 console 2>> data1.log

Check existing account
> eth.accounts
[]
> personal.newAccount("vizeet")
"0x9dfbb82464a0cf33bb37df9e5dbb7b2b8740c51b"
> eth.coinbase
"0x9dfbb82464a0cf33bb37df9e5dbb7b2b8740c51b"
> eth.getBalance(eth.coinbase)
0
> miner.start()
null
> eth.getBalance(eth.coinbase)
21000000000000000000
> miner.stop()
true
> eth.getBalance(eth.coinbase)
54000000000000000000

On Terminal 2: initialize datadir and create genesis block
geth --datadir="data2" --maxpeers=1 init genesis.json

start console
geth --datadir="data2" -verbosity 6 --ipcdisable --port 30302 --rpcport 8102 --networkid 123 console 2>> data2.log
Welcome to the Geth JavaScript console!

check peer count on both terminals
> web3.net.peerCount
0
get endpoint of node 1 (Terminal 1)
> admin.nodeInfo.enode
"enode://eb1b16f13048382a797586f4390d8013629ace1d76be7bce207138fd58c24a8c8cb3c5d78077765f1c34e13a75119e12b094535f19b277dc34873d3dcbf4609e@[::]:30301"

Add Peer in Terminal 2
> admin.addPeer("enode://eb1b16f13048382a797586f4390d8013629ace1d76be7bce207138fd58c24a8c8cb3c5d78077765f1c34e13a75119e12b094535f19b277dc34873d3dcbf4609e@[::]:30301")
true

check peer on both terminal 
> web3.net.peerCount
1

Add Account Terminal 2
> eth.accounts
[]
> personal.newAccount("user2")
"0xb9ce2c14e0e4ca784b08bda2c7b42ff533834e48"
> eth.accounts
["0xb9ce2c14e0e4ca784b08bda2c7b42ff533834e48"]

Mine eth into new account (Terminal 2)
> eth.getBalance(eth.coinbase)
0
>  miner.start()
null
> eth.getBalance(eth.coinbase)
33000000000000000000
> miner.stop()
true
> eth.getBalance(eth.coinbase)
54000000000000000000

Save checkBalances.js
----------
function checkAllBalances() {
  web3.eth.getAccounts(function(err, accounts) {
   accounts.forEach(function(id) {
    web3.eth.getBalance(id, function(err, balance) {
     console.log("" + id + ":\tbalance: " + web3.fromWei(balance, "ether") + " ether");
   });
  });
 });
};
----------
> loadScript('checkBalances.js')
true
> checkAllBalances()
0x9dfbb82464a0cf33bb37df9e5dbb7b2b8740c51b:	balance: 54 ether
undefined

Greeter.sol
------------------
contract mortal {
    /* Define variable owner of the type address */
    address owner;

    /* This function is executed at initialization and sets the owner of the contract */
    function mortal() { owner = msg.sender; }

    /* Function to recover the funds on the contract */
    function kill() { if (msg.sender == owner) selfdestruct(owner); }
}

contract greeter is mortal {
    /* Define variable greeting of the type string */
    string greeting;

    /* This runs when the contract is executed */
    function greeter(string _greeting) public {
        greeting = _greeting;
    }

    /* Main function */
    function greet() constant returns (string) {
        return greeting;
    }
}
----------------

compile this with solc 
solc -o target --bin --abi Greeter.sol

type JS statements in console
var abi = <abi from solc>;
 var myContract = eth.contract(abi); 
 var bytecode = '0x' + <bytecode from solc>;
 var txDeploy = {from:eth.coinbase, data: bytecode, gas: 1000000}; 
 var myContractPartialInstance = myContract.new(txDeploy); 

 // Mine block containing transaction...

 var myContractInstance = myContract.at(myContractPartialInstance.address);
 
 Mine block again to get the contract into block
 > miner.start()
null
> eth.getBalance(eth.coinbase)
84000000000000000000
> miner.stop()
true
> eth.getBalance(eth.coinbase)
111000000000000000000

Check if contract is mined
> myContractInstance.address
"0xbd6f768b23a5e305a83c19a0c8f1c767a53a2548"

Running greet
> myContractInstance.greet()
""
