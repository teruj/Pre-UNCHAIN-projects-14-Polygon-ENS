### ðãã¡ã¤ã³ã¬ã³ã¼ãã®æ´æ°

ã¹ãã¼ãã³ã³ãã©ã¯ãã§ã¯ãã¡ã¤ã³ã¬ã³ã¼ããæ´æ°å¯è½ã«ãã¾ããããã¢ããªã§ãã®æ©è½ãæ§ç¯ãã¦ãã¾ããããããå®è£ãã¦ã¿ã¾ãããã`App.js`ã®`mintDomain`é¢æ°ã®æ¬¡ã«ãã®é¢æ°ãè¿½å ãã¾ãããã

```jsx
const updateDomain = async () => {
  if (!record || !domain) { return }
  setLoading(true);
  console.log("Updating domain", domain, "with record", record);
    try {
      const { ethereum } = window;
      if (ethereum) {
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const contract = new ethers.Contract(CONTRACT_ADDRESS, contractAbi.abi, signer);

        let tx = await contract.setRecord(domain, record);
        await tx.wait();
        console.log("Record set https://mumbai.polygonscan.com/tx/"+tx.hash);

        fetchMints();
        setRecord('');
        setDomain('');
      }
    } catch(error) {
      console.log(error);
    }
  setLoading(false);
}
```

ããã§ã¯ç¹ã«æ°ãããããã¯ã¯ãªãã§ããããããã¯`mintDomain`é¢æ°ã¨å±éããã¨ãããå¤ãã§ãã­ã çããã¯ããã¾ã§ã®å­¦ç¿ã§ååç¿çãã¦ããã®ã§çè§£ã§ããã§ãããð

ãããå®éã«å¼ã³åºãã«ã¯ã`renderInputForm`é¢æ°ã«ããã«ããã¤ãã®å¤æ´ãå ãã¦`Set record`ãã¿ã³ãè¡¨ç¤ºããå¿è¦ãããã¾ãã

ã¾ããç¶æå¤æ°ãä½¿ç¨ãã¦ãEditã¢ã¼ãã§ãããã©ãããæ¤åºãã¾ãã

ä¸ã®ã³ã¼ãã§ã¯`editing`ã¨ãã¦å®ç¾©ãã¦ãã¾ãã



```jsx
  const App = () => {
  // æ°ããç¶æå¤æ°ãå®ç¾©ãã¦ãã¾ããããã¾ã§ã®ãã®ã®ä¸ã«è¿½å ãã¾ãããã
  const [editing, setEditing] = useState(false);
  const [loading, setLoading] = useState(false);

  // renderInputFormé¢æ°ãå¤æ´ãã¾ãã
  const renderInputForm = () =>{
    if (network !== 'Polygon Mumbai Testnet') {
      return (
        <div className="connect-wallet-container">
          <p>Please connect to Polygon Mumbai Testnet</p>
          <button className='cta-button mint-button' onClick={switchNetwork}>Click here to switch</button>
        </div>
      );
    }

    return (
      <div className="form-container">
        <div className="first-row">
          <input
            type="text"
            value={domain}
            placeholder='domain'
            onChange={e => setDomain(e.target.value)}
          />
          <p className='tld'> {tld} </p>
        </div>

        <input
          type="text"
          value={record}
          placeholder='whats ur ninja power?'
          onChange={e => setRecord(e.target.value)}
        />
          {/* editing å¤æ°ã true ã®å ´åã"Set record" ã¨ "Cancel" ãã¿ã³ãè¡¨ç¤ºãã¾ãã */}
          {editing ? (
            <div className="button-container">
              {/* updateDomainé¢æ°ãå¼ã³åºãã¾ãã */}
              <button className='cta-button mint-button' disabled={loading} onClick={updateDomain}>
                Set record
              </button>  
              {/* editing ã false ã«ãã¦Editã¢ã¼ãããæãã¾ãã*/}
              <button className='cta-button mint-button' onClick={() => {setEditing(false)}}>
                Cancel
              </button>  
            </div>
          ) : (
            // editing å¤æ°ã true ã§ãªãå ´åãMint ãã¿ã³ãä»£ããã«è¡¨ç¤ºããã¾ãã
            <button className='cta-button mint-button' disabled={loading} onClick={mintDomain}>
              Mint
            </button>  
          )}
      </div>
    );
  }
```

### ðãã³ãã¬ã³ã¼ããåå¾ãã

ããã¦ã¼ã¶ã¼ãè³¼å¥ãã¦ããããã¡ã¤ã³ãä¸çã«å¬éã§ãã¾ãã

ãããè¡ãããã®é¢æ°ãã³ã³ãã©ã¯ãã«è¿½å ãã¾ããã

```jsx
// ç¶æãç®¡çãã mints ãå®ç¾©ãã¾ããåæç¶æã¯ç©ºã®éåã§ãã
const [mints, setMints] = useState([]);

// mintDomainé¢æ°ã®ãã¨ã«è¿½å ããã®ãããããããã§ãããã
const fetchMints = async () => {
  try {
    const { ethereum } = window;
    if (ethereum) {
      // ããçè§£ã§ãã¦ãã¾ãã­ã
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const contract = new ethers.Contract(CONTRACT_ADDRESS, contractAbi.abi, signer);
        
      // ãã¹ã¦ã®ãã¡ã¤ã³ãåå¾ãã¾ãã
      const names = await contract.getAllNames();
        
      // ãã¼ã ãã¨ã«ã¬ã³ã¼ããåå¾ãã¾ãããããã³ã°ã®å¯¾å¿ãçè§£ãã¾ãããã
      const mintRecords = await Promise.all(names.map(async (name) => {
      const mintRecord = await contract.records(name);
      const owner = await contract.domains(name);
      return {
        id: names.indexOf(name),
        name: name,
        record: mintRecord,
        owner: owner,
      };
    }));

    console.log("MINTS FETCHED ", mintRecords);
    setMints(mintRecords);
    }
  } catch(error){
    console.log(error);
  }
}

// currentAccount, network ãå¤ãããã³ã«å®è¡ããã¾ãã
useEffect(() => {
  if (network === 'Polygon Mumbai Testnet') {
    fetchMints();
  }
}, [currentAccount, network]);
```

1. ã³ã³ãã©ã¯ããããã¹ã¦ã®ãã¡ã¤ã³å
2. åå¾ããåãã¡ã¤ã³ã®ã¬ã³ã¼ã
3. åå¾ããåãã¡ã¤ã³ã®ææèã®ã¢ãã¬ã¹

ããããéåã«å¥ããéåã`mints`ã¨ãã¦è¨­å®ãã¾ãã

`mintDomain`é¢æ°ã®ä¸é¨ã«ãè¿½å ããã®ã§ããã¡ã¤ã³ãèªåã§ä½æããã¨ã¢ããªãæ´æ°ããã¾ãã ãã©ã³ã¶ã¯ã·ã§ã³ããã¤ãã³ã°ããã¦ãããã¨ãç¢ºèªããããã«2ç§å¾ã£ã¦ãã¾ãã ããã§ãã¦ã¼ã¶ã¼ã¯èªåã®ãã³ãããªã¢ã«ã¿ã¤ã ã§è¦ããã¨ãã§ãã¾ãã

```jsx
const mintDomain = async () => {
  // domain ã®å­å¨ç¢ºèªã§ãã
  if (!domain) { return }
  // ãã¡ã¤ã³ãç­ãããå ´åã¢ã©ã¼ããåºãã¾ãã
  if (domain.length < 3) {
    alert('Domain must be at least 3 characters long');
    return;
  }

  // ãã¡ã¤ã³ã®æå­æ°ã§ä¾¡æ ¼ãè¨ç®ãã¾ãã
  // 3æå­ = 0.5 MATIC, 4æå­ = 0.3 MATIC, 5æå­ä»¥ä¸ = 0.1 MATIC
  // ãèªåã§è¨­å®ãå¤ãã¦ãæ§ãã¾ããããç¾å¨ã¦ã©ã¬ããã«ã¯å°éãããªãã¯ãã§ãããã
  const price = domain.length === 3 ? '0.5' : domain.length === 4 ? '0.3' : '0.1';
  console.log("Minting domain", domain, "with price", price);
  try {
      const { ethereum } = window;
      if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const contract = new ethers.Contract(CONTRACT_ADDRESS, contractAbi.abi, signer);

      console.log("Going to pop wallet now to pay gas...")
          let tx = await contract.register(domain, {value: ethers.utils.parseEther(price)});
      // ãã©ã³ã¶ã¯ã·ã§ã³ãå¾ã¡ã¾ã
      const receipt = await tx.wait();

      // ãã©ã³ã¶ã¯ã·ã§ã³ã®æåã®ç¢ºèªã§ãã
      if (receipt.status === 1) {
        console.log("Domain minted! https://mumbai.polygonscan.com/tx/"+tx.hash);
        
        // domain ã® record ãã»ãããã¾ãã
        tx = await contract.setRecord(domain, record);
        await tx.wait();

        console.log("Record set! https://mumbai.polygonscan.com/tx/"+tx.hash);
        
        // fetchMintsé¢æ°å®è¡å¾2ç§å¾ã¡ã¾ãã
        setTimeout(() => {
          fetchMints();
        }, 2000);

        setRecord('');
        setDomain('');
      } else {
        alert("Transaction failed! Please try again");
      }
      }
    } catch(error) {
      console.log(error);
    }
}
```

### ð§ãã³ãããããã¡ã¤ã³ãã¬ã³ããªã³ã°ãã

ã»ã¼å®äºã§ããã¬ã³ããªã³ã°é¢æ°ãè¨­å®ãã¾ãã

```jsx
// ä»ã®ã¬ã³ããªã³ã°é¢æ°ã®æ¬¡ã«è¿½å ãã¾ãããã
const renderMints = () => {
  if (currentAccount && mints.length > 0) {
    return (
      <div className="mint-container">
        <p className="subtitle"> Recently minted domains!</p>
        <div className="mint-list">
          { mints.map((mint, index) => {
            return (
              <div className="mint-item" key={index}>
                <div className='mint-row'>
                  <a className="link" href={`https://testnets.opensea.io/assets/mumbai/${CONTRACT_ADDRESS}/${mint.id}`} target="_blank" rel="noopener noreferrer">
                    <p className="underlined">{' '}{mint.name}{tld}{' '}</p>
                  </a>
                  {/* mint.owner ã currentAccount ãªã edit ãã¿ã³ãè¿½å ãã¾ãã */}
                  { mint.owner.toLowerCase() === currentAccount.toLowerCase() ?
                    <button className="edit-button" onClick={() => editRecord(mint.name)}>
                      <img className="edit-icon" src="https://img.icons8.com/metro/26/000000/pencil.png" alt="Edit button" />
                    </button>
                    :
                    null
                  }
                </div>
          <p> {mint.record} </p>
        </div>)
        })}
      </div>
    </div>);
  }
};

// edit ã¢ã¼ããè¨­å®ãã¾ãã
const editRecord = (name) => {
  console.log("Editing record for", name);
  setEditing(true);
  setDomain(name);
}
```

`mints.map`é¨åã§ã¯ã`mints`éååã®åã¢ã¤ãã ãåå¾ãããã®ããã®HTMLãã¬ã³ããªã³ã°ãã¾ãã å®éã®HTMLã®é ç®ã®å¤ã`mint.name`ã¨`mint.id`ã§ä½¿ç¨ãã¾ãããã®é¢æ°ã¯ãä»ã®ãã¹ã¦ã®ã¬ã³ããªã³ã°é¢æ°ã§å¼ã³åºããã¨ãã§ãã¾ãã

ã¬ã³ããªã³ã°ã»ã¯ã·ã§ã³ã¯ã¾ã¨ããã°æ¬¡ã®ãããªã­ã¸ãã¯ã§ãã

```jsx
{!currentAccount && renderNotConnectedContainer()}
{currentAccount && renderInputForm()}
{mints && renderMints()}
```

è¤æ°ãã³ãããã¦ããå ´åãç»é¢ã¯æ¬¡ã®ãããªãã®ã«ãªãã¾ãã

![](/public/images/14-Polygon-ENS/section-4/4_1_4.png)

ããæãã§ãã­ã

éç­ã®ç®æãã¯ãªãã¯ããã¨ç·¨éã§ãã¾ãã



ææããåãã¡ã¤ã³ã®ã¬ã³ã¼ããç·¨éã§ãã¾ãã

![](/public/images/14-Polygon-ENS/section-4/4_1_5.png)

æ´æ°ãããã¨ããã¨å½ç¶ãã©ã³ã¶ã¯ã·ã§ã³ãçºçãã¾ãã

### ðââï¸ è³ªåãã

ããã¾ã§ã®ä½æ¥­ã§ä½ãããããªããã¨ãããå ´åã¯ãDiscord ã® `#section-4` ã§è³ªåããã¦ãã ããã

ãã«ããããã¨ãã®ãã­ã¼ãåæ»ã«ãªãã®ã§ãã¨ã©ã¼ã¬ãã¼ãã«ã¯ä¸è¨ã® 3 ç¹ãè¨è¼ãã¦ãã ãã â¨

```
1. è³ªåãé¢é£ãã¦ããã»ã¯ã·ã§ã³çªå·ã¨ã¬ãã¹ã³çªå·
2. ä½ããããã¨ãã¦ããã
3. ã¨ã©ã¼æãã³ãã¼&ãã¼ã¹ã
4. ã¨ã©ã¼ç»é¢ã®ã¹ã¯ãªã¼ã³ã·ã§ãã
```

---
ãç²ãæ§ã§ãããæ¬¡ã®ã¬ãã¹ã³ã«é²ã¿ã¾ãããï¼ï¼