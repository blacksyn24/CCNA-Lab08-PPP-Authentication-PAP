# 🔐 Lab 8 — PPP Authentication Using PAP

![Cisco](https://img.shields.io/badge/Cisco-CCNA-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/101%20Labs-Lab%208-purple?style=for-the-badge)
![PPP](https://img.shields.io/badge/PPP-PAP%20Authentication-red?style=for-the-badge)

---

## 📋 Description

Huitième lab du livre **101 Labs Cisco CCNA (200-301)**.
L'objectif est de configurer l'authentification **PPP PAP**
entre deux routeurs Cisco sur un lien Serial. PAP envoie
les credentials en clair mais reste une compétence
fondamentale pour l'examen CCNA.

### Objectifs
- ✅ Configurer les **hostnames** sur R1 et R2
- ✅ Activer les interfaces **Serial Se0/0/0**
- ✅ Configurer le **clock rate 768 Kbps** sur R2
- ✅ Configurer l'encapsulation **PPP**
- ✅ Configurer l'authentification **PAP**
- ✅ Observer l'authentification PAP via **debug**

---

## 📖 Théorie

### PAP vs CHAP
```
PAP (Password Authentication Protocol)
→ Envoie username + password en CLAIR ❌
→ Moins sécurisé
→ 2 étapes : envoi + acceptation

CHAP (Challenge Handshake Auth Protocol)
→ Mot de passe chiffré ✅
→ Plus sécurisé
→ 3 étapes : challenge + réponse + acceptation
```

### Comment PAP fonctionne
```
R1 ──► "Je suis R1, mdp = PAP" ──► R2
R2 vérifie : username R1 + password PAP ✅
R2 ──► "Accès autorisé !" ──► R1

R2 ──► "Je suis R2, mdp = PAP" ──► R1
R1 vérifie : username R2 + password PAP ✅
R1 ──► "Accès autorisé !" ──► R2
```

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | Cisco 2911 | R1 | DTE |
| 🔀 Routeur | Cisco 2911 | R2 | DCE |

---

## 🗺️ Topologie

```
[R1]──Se0/0/0──────────────Se0/0/0──[R2]
(DTE) 10.0.254.1        10.0.254.2  (DCE)
              10.0.254.0/28
              PPP + PAP Authentication
              Clock rate : 768 Kbps
```

<img width="1920" height="1080" alt="TopoLab8" src="https://github.com/user-attachments/assets/b9821e2e-6730-4a17-8626-56ff87bdba17" />


---

## 📊 Plan d'adressage

| Interface | IP | Réseau | Rôle | Routeur |
|-----------|-----|--------|------|---------|
| Se0/0/0 | 10.0.254.1 | 10.0.254.0/28 | DTE | R1 |
| Se0/0/0 | 10.0.254.2 | 10.0.254.0/28 | DCE | R2 |

---

## ⚙️ Configuration complète

### 🔧 Task 1 — Hostnames

```cisco
enable
configure terminal
hostname R1
end
```

```cisco
enable
configure terminal
hostname R2
end
```

---

### 🔧 Task 2 — Serial + Clock Rate 768Kbps

```cisco
! R1 (DTE)
enable
configure terminal
interface serial 0/0/0
no shutdown
exit
end
```

```cisco
! R2 (DCE)
enable
configure terminal
interface serial 0/0/0
clock rate 768000
no shutdown
exit
end
```

```cisco
! Vérifier DCE/DTE
R2# show controllers serial 0/0/0
R1# show controllers serial 0/0/0
```

---

### 🔧 Task 3 — PPP + IP

```cisco
! R1
enable
configure terminal
interface serial 0/0/0
encapsulation ppp
ip address 10.0.254.1 255.255.255.240
no shutdown
exit
end
write
```

```cisco
! R2
enable
configure terminal
interface serial 0/0/0
encapsulation ppp
ip address 10.0.254.2 255.255.255.240
clock rate 768000
no shutdown
exit
end
write
```

---

### 🔧 Task 4 — Vérifier PPP + Ping

```cisco
R1# show interfaces serial 0/0/0
R1# ping 10.0.254.2
R2# ping 10.0.254.1
```

---

### 🔧 Task 5 — Usernames

```cisco
! Sur R1
configure terminal
username R2 password PAP
exit
```

```cisco
! Sur R2
configure terminal
username R1 password PAP
exit
```

---

### 🔧 Task 6 — PAP Authentication

```cisco
! Sur R1
configure terminal
interface serial 0/0/0
ppp authentication pap
ppp pap sent-username R1 password PAP
exit
end
write
```

```cisco
! Sur R2
configure terminal
interface serial 0/0/0
ppp authentication pap
ppp pap sent-username R2 password PAP
exit
end
write
```

---

### 🔧 Task 7 — Debug PAP

```cisco
R1# debug ppp authentication
```

```cisco
R1(config)# interface serial 0/0/0
R1(config-if)# shutdown
R1(config-if)# no shutdown
```

**Résultat attendu :**
```
PAP: I AUTH-REQ  ← R2 envoie credentials
PAP: O AUTH-ACK  ← R1 accepte ✅
PAP: I AUTH-ACK  ← R2 accepte ✅
```

```cisco
R1# no debug all
```

---

## 🧪 Tests finaux

```cisco
R1# ping 10.0.254.2  ✅
R2# ping 10.0.254.1  ✅
R1# show ppp all
R1# show interfaces serial 0/0/0
```

---

## 💡 Points clés

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `username R2 password PAP` | Créer compte local pour R2 |
| `ppp authentication pap` | Activer auth PAP |
| `ppp pap sent-username R1 password PAP` | Envoyer credentials |
| `debug ppp authentication` | Voir auth en temps réel |
| `no debug all` | Désactiver debug |
| `show ppp all` | Voir état PPP |

---

## 📊 Comparatif Labs PPP

| | Lab 7 (PPP) | Lab 8 (PAP) |
|---|---|---|
| **Encapsulation** | PPP | PPP |
| **Auth** | ❌ Aucune | ✅ PAP |
| **Sécurité** | Faible | Faible (clair) |
| **Clock rate** | 512 Kbps | 768 Kbps |
| **Debug** | `debug ppp negotiation` | `debug ppp authentication` |
| **Username** | ❌ Non | ✅ Oui |

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
