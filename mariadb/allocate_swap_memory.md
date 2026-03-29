## Add 16GB Swap on Ubuntu 24 — Step by Step

---

**Step 1 — Turn off existing swap**
```bash
sudo swapoff /swap.img
```

**Step 2 — Delete old swap file**
```bash
sudo rm /swap.img
```

**Step 3 — Create new 16GB swap file**
```bash
sudo fallocate -l 16G /swap.img
```

**Step 4 — Set permissions**
```bash
sudo chmod 600 /swap.img
```

**Step 5 — Format as swap**
```bash
sudo mkswap /swap.img
```

**Step 6 — Enable swap**
```bash
sudo swapon /swap.img
```

**Step 7 — Reduce swappiness**
```bash
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

**Step 8 — Verify**
```bash
free -h
swapon --show
```

---

✅ You should see **16GB swap** in the output. Done!
