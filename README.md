# LocalSupa

Boş port kontrolu:

'''bash
ss -ltn | grep 55432 || echo "55432 boş"
'''

supa baslatma

sudo -u postgres psql -c "show config_file;"


port değiştirmek: (ctrl+w ile "port" aratın ve değiştirin"

sudo nano /etc/postgresql/14/main/postgresql.conf


