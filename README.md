<h1>&nbsp;&nbsp;&nbsp;&nbsp;Sistem-IoT-de-monitorizare-a-temperaturii-si-umiditatii-cu-criptare-si-recompensare-prin-blockchain </h1>
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Proiectul folosește microcontrollerul STM32F103C8T6 pentru achiziția datelor de la senzorii de temperatură și umiditate. Informațiile sunt procesate local, criptate cu AES-GCM și transmise prin protocolul HTTPS/TLS către o instanță cloud. În cloud, datele sunt verificate și integrate într-un sistem blockchain, care oferă recompense în tokeni ERC-20.</h3>

<img width="651" height="141" alt="Untitled Diagram drawio" src="https://github.com/user-attachments/assets/9b996a91-182d-496a-a880-e804fb7a6fb7" />

:one: **Senzor Temperatură**

- Măsoară temperatura mediului o dată pe minut  
- Trimite valoarea către STM32.

---

:two: **Senzor Umiditate**

- Măsoară umiditatea relativă a aerului.  
- Trimite valoarea către STM32.

---

🟨 :three: **STM32 (microcontroller)** — Este „creierul local” al sistemului.

1. Citește datele de la senzori (temperatură și umiditate).  
2. Preprocesează valorile (ex.: rotunjire, filtrare, calibrare).  
3. Construiește un pachet de date (mesaj JSON).  
4. Criptează conținutul (payload-ul) cu **AES-GCM** folosind o cheie unică (`K_device`):  
   - Generează un IV (Initialization Vector) sau *nonce* — un număr aleator unic pentru fiecare criptare.  
   - Produce **ciphertext** (date criptate) și **tag** (pentru verificarea integrității).  
5. Trimite mesajul criptat prin **HTTPS/TLS** către instanța din cloud.

---

🟧 :four: **Instanța Cloud** — Este „operatorul central” al sistemului.  
Face toate funcțiile de procesare, validare și legătura cu blockchain-ul.

1. Primește mesajul HTTPS (TLS terminat aici).  
2. Verifică integritatea pachetului:  
   - Schema corectă (toate câmpurile există).  
   - `seq` în ordine (anti-replay).  
   - `ts` în fereastra de timp (±5 min).  
3. *(Opțional)* Calculează hash-ul conținutului criptat `SHA256(ciphertext)` fără decriptare.  
4. Stochează mesajul în baza de date (**ciphertext** + metadate).  
5. Contorizează activitatea fiecărui senzor:  
   - La fiecare 10 citiri → 1 token reward.  
6. Trimite tranzacția către blockchain:  
   - Apelează funcția `mint()` sau `transfer()` a contractului ERC-20 pe testnet.  
   - Include `sensor_id`, `amount=1`, `hash(ciphertext)` (opțional pentru audit).  
7. Primește confirmarea blockchainului și salvează `tx_hash` în DB.

---

🟥 :five: **Blockchain (rețea testnet)**

- Primește tranzacția `mint` sau `transfer` de la instanța din cloud.  
- Confirmă și înregistrează tranzacția în lanț.  
- Trimite tokenii către walletul asociat senzorului (sau operatorului).  
- *(Opțional)* păstrează în log evenimentul `hash(ciphertext)` ca dovadă criptografică.

---

🟪 :six: **Wallet de test**

- Primește tokenii generați ca recompensă.  
- Poți vizualiza balanța tokenilor în **Metamask** (pe rețeaua testnet).  
- Servește ca dovadă că datele reale (de la senzori) au fost procesate corect și recompensate.
