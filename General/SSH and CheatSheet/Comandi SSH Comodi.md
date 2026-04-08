### 🖥️ 1. SSH e Gestione Server

| **Comando**                              | **Descrizione**                                                  |
| ---------------------------------------- | ---------------------------------------------------------------- |
| `ssh fabiopsh@192.168.1.32`              | Connette il tuo terminale al server remoto.                      |
| `exit`                                   | Chiude la connessione SSH e torna al tuo PC.                     |
| `ip a`                                   | Mostra le interfacce di rete e gli indirizzi IP del server.      |
| `sudo apt update && sudo apt upgrade -y` | Cerca e installa tutti gli aggiornamenti di sistema disponibili. |
| `sudo reboot`                            | Riavvia fisicamente il server.                                   |
| `sudo shutdown now`                      | Spegne definitivamente il server (richiede accensione manuale).  |

---

### 🐳 2. Docker: Gestione Container e Sistema

|**Comando**|**Descrizione**|
|---|---|
|`docker ps`|Elenca tutti i container attualmente in esecuzione.|
|`docker ps -a`|Elenca **tutti** i container (inclusi quelli fermi o andati in crash).|
|`docker start <nome_container>`|Accende un container specifico.|
|`docker stop <nome_container>`|Spegne un container specifico in modo sicuro.|
|`docker logs <nome_container>`|Stampa a schermo i log di errore o output del container.|
|`docker logs -f <nome_container>`|Mostra i log in tempo reale (premi `CTRL+C` per uscire).|
|`docker stats`|Mostra il consumo di RAM e CPU dei container in tempo reale.|
|`docker system prune`|Elimina container fermi, reti non usate e vecchie immagini (libera spazio).|

---

### 📦 3. Docker Compose: Gestione Progetti

_(Nota: Questi comandi vanno eseguiti dentro la cartella dove si trova il file `docker-compose.yml`)_

| **Comando**                    | **Descrizione**                                                                |
| ------------------------------ | ------------------------------------------------------------------------------ |
| `docker compose up -d`         | Crea e avvia tutti i servizi del progetto in background (Detached).            |
| `docker compose stop`          | Ferma temporaneamente il progetto (i container rimangono intatti).             |
| `docker compose down`          | Spegne e distrugge i container e la rete interna (NON cancella i volumi/dati). |
| `docker compose build`         | Ricostruisce le immagini locali partendo dai `Dockerfile`.                     |
| `docker compose up -d --build` | Comando rapido: ricostruisce le immagini e avvia/aggiorna i container.         |
| `docker compose logs -f`       | Mostra i log combinati di tutti i servizi del progetto in tempo reale.         |

---
GalimbertiCoinqui0103!