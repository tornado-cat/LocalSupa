# LocalSupa

Boş port kontrolu:

'''bash
ss -ltn | grep 55432 || echo "55432 boş"
'''

supa baslatma

sudo -u postgres psql -c "show config_file;"


port değiştirmek: (ctrl+w ile "port" aratın ve değiştirin"

sudo nano /etc/postgresql/14/main/postgresql.conf
sudo systemctl restart postgresql

kontrol:

ss -ltn | grep 55432


şifre ayarlama:

sudo -u postgres psql -p 55432
ALTER USER postgres PASSWORD 'sifre';
\q (çikis yapar)

database için url ayarlama

export DATABASE_URL='postgresql://postgres:123@localhost:55432/postgres'

restore.pty oluşturma 

cat > restore.py <<'PY'
import os, gzip, shutil, subprocess
from pathlib import Path

BACKUP_GZ = Path("/mnt/data/db_cluster-27-08-2025@05-59-17.backup.gz")
DB_URL = os.environ.get("DATABASE_URL", "").strip()

def run(cmd):
    print("RUN:", " ".join(cmd))
    p = subprocess.run(cmd, text=True, capture_output=True)
    if p.stdout: print(p.stdout)
    if p.stderr: print(p.stderr)
    if p.returncode != 0:
        raise SystemExit(p.returncode)

def gunzip(src: Path) -> Path:
    dst = src.with_suffix("")  # drop .gz
    if dst.exists():
        dst = src.parent / (src.stem + ".unzipped")
    print(f"Unzipping {src} -> {dst}")
    with gzip.open(src, "rb") as f_in, open(dst, "wb") as f_out:
        shutil.copyfileobj(f_in, f_out)
    return dst

def detect(unzipped: Path) -> str:
    with open(unzipped, "rb") as f:
        head = f.read(64)
    if head.startswith(b"PGDMP"):
        return "custom"
    txt = head.decode("utf-8", errors="ignore").lstrip().upper()
    if txt.startswith(("--", "CREATE", "SET", "BEGIN", "ALTER", "INSERT")):
        return "sql"
    return "custom"

def main():
    if not DB_URL:
        raise SystemExit("DATABASE_URL boş. export DATABASE_URL=... ile set et.")
    if not BACKUP_GZ.exists():
        raise SystemExit(f"Backup yok: {BACKUP_GZ}")
    unz = gunzip(BACKUP_GZ)
    fmt = detect(unz)
    print("Detected:", fmt)

    if fmt == "sql":
        run(["psql", DB_URL, "-v", "ON_ERROR_STOP=1", "-f", str(unz)])
    else:
        run(["pg_restore", "--clean", "--if-exists", "--no-owner", "--no-privileges",
             "-d", DB_URL, str(unz)])
    print("OK")

if __name__ == "__main__":
    main()

restore.py çalıştırma:
ls -la restore.py
python3 restore.py
