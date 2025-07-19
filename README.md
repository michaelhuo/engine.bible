Engine.bible
Engine.bible is a next-generation Bible search engine API powering https://engine.bible. Built with Python FastAPI and MariaDB, it provides fast, reliable access to Bible verses in multiple translations, including public domain texts (CUVS, CUVT, KJV, Pinyin) and the authorized NASB. The API integrates with the bible.world MediaWiki for rich contextual content, such as commentaries and encyclopedic entries. Designed for developers, ministries, and Christian applications, it offers versioned endpoints (e.g., /v1/search) for querying verses, liking verses, and accessing wiki data, with a focus on performance, scalability, and ease of use.
This project is decoupled from bible.world, which hosts the MediaWiki instance, ensuring modularity and independent updates. The API supports multiple languages (Simplified Chinese, Traditional Chinese, English) and provides links to external translations (e.g., ESV, NCVS, LCVS) for broader accessibility.
Features

Verse Search: Query Bible verses by reference (e.g., "John 3:16") or keywords, with support for single or multi-verse searches and an extend option (±n verses).
Multi-Language Support: Book names in Simplified Chinese (CUVS), Traditional Chinese (CUVT), and English (KJV, NASB).
Wiki Integration: Fetch content from bible.world MediaWiki for commentaries and encyclopedia entries.
Like Functionality: Increment a "likes" counter for verses, stored in the MariaDB database.
API Versioning: Versioned endpoints (e.g., /v1) for future-proofing and backward compatibility.
Supported Translations:
Public Domain: Chinese Union Version Simplified (CUVS), Chinese Union Version Traditional (CUVT), King James Version (KJV), Pinyin.
Authorized: New American Standard Bible (NASB).
External Links: ESV, NCVS, LCVS, CCSB, CLBS, CKJVS, CKJVT, UKJV, KJV1611, BBE.


Scalable Backend: Uses FastAPI for high-performance async API handling and MariaDB for reliable data storage.

Installation
Prerequisites

OS: Ubuntu 24.04 LTS (or compatible Linux distribution).
Dependencies: Python 3.12, MariaDB 10.11.13, Apache2.
Database: MariaDB database named bible with tables bible_books and bible_search (or bible_multi_search).

Setup

Clone the Repository:
git clone https://github.com/michaelhuo/engine.bible.git
cd engine.bible


Install Dependencies:
sudo apt update
sudo apt install python3-venv python3-pip mariadb-server libmariadb-dev python3-dev build-essential
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt


Configure Database:Create config.py with your MariaDB credentials:
DATABASE_URL = "mysql+pymysql://bible:9410228Bible!@localhost/bible"
WIKI_BASE_URL = "https://bible.world/w"
MAX_RECORD_COUNT = 500
MAX_KEYWORDS = 10

Verify database access:
mysql -u bible -p9410228Bible! -e "SELECT * FROM bible_books LIMIT 1;" bible


Run Locally:
source venv/bin/activate
uvicorn main:app --reload --host 0.0.0.0 --port 8000

Access http://localhost:8000/docs for the interactive Swagger UI to test endpoints.

Configure Apache2:Create /etc/apache2/sites-available/engine.bible.conf:
<VirtualHost *:80>
    ServerName engine.bible
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
    ErrorLog /data/logs/bible.world/error.log
    CustomLog /data/logs/bible.world/access.log combined
</VirtualHost>

Enable and reload:
sudo a2ensite engine.bible.conf
sudo a2enmod proxy proxy_http
sudo systemctl reload apache2


Set Up Systemd Service:Create /etc/systemd/system/fastapi.service:
[Unit]
Description=FastAPI Engine.bible
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/opt/www/engine.bible
Environment="PATH=/opt/www/engine.bible/venv/bin"
ExecStart=/opt/www/engine.bible/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target

Enable and start:
sudo systemctl daemon-reload
sudo systemctl start fastapi
sudo systemctl enable fastapi



Usage
API Endpoints

GET /v1/search: Search for Bible verses.

Query Parameters:
q (required): Verse reference (e.g., John 3:16) or keywords.
l (optional): Language (cn, tw, en). Default: cn.
m (optional): Multi-verse search (0 for single, 1 for multi). Default: 0.
e (optional): Extend verses before/after (0-5). Default: 0.


Example:curl "https://engine.bible/v1/search?q=John+3:16&l=en&e=3"

Response:{
  "verses": [
    {
      "book": 43,
      "chapter": 3,
      "verse": 16,
      "text_cuvs": "...",
      "text_cuvt": "...",
      "text_kjv": "For God so loved the world...",
      "text_nasb": "...",
      "likes": 0,
      "external_links": {
        "Blue Letter Bible": "http://www.blueletterbible.org/Bible.cfm?b=John&c=3&v=16",
        "Bible Gateway ESV": "http://www.biblegateway.com/passage/?version=ESV&search=John+3:16"
        // Other links
      }
    }
  ],
  "message": "Found 1 verses"
}




GET /v1/wiki: Search bible.world MediaWiki content.

Query Parameters:
q (required): Wiki query (e.g., Jesus).
p (optional): Page number (1+). Default: 1.


Example:curl "https://engine.bible/v1/wiki?q=Jesus"

Response:{
  "text": "Wiki content about Jesus..."
}




POST /v1/like: Increment likes for a verse.

Query Parameters:
book (required): Book ID (1-66).
chapter (required): Chapter number.
verse (required): Verse number.


Example:curl -X POST "https://engine.bible/v1/like?book=43&chapter=3&verse=16"

Response:{
  "message": "Verse liked successfully"
}





Supported Bible Translations

Hosted (in database):
Chinese Union Version Simplified (CUVS, public domain)
Chinese Union Version Traditional (CUVT, public domain)
King James Version (KJV, public domain)
New American Standard Bible (NASB, authorized by The Lockman Foundation)
Pinyin (public domain)


Linked (external):
English Standard Version (ESV, via Bible Gateway)
New Chinese Version (NCVS)
Lü Zhenzhong (LCVS)
Catholic Study Bible (CCSB)
Contemporary Bible (CLBS)
Chinese King James Version Simplified (CKJVS)
Chinese King James Version Traditional (CKJVT)
Updated King James Version (UKJV)
King James Version 1611 (KJV1611)
Bible in Basic English (BBE)



License
This project is licensed under a proprietary license requiring explicit authorization from the author. See LICENSE for details.
Bible text data includes:

Public domain: CUVS, CUVT, KJV, Pinyin.
NASB: Used with permission from The Lockman Foundation.
External translations (e.g., ESV) are linked to third-party websites and subject to their respective licenses.

Contributing
Contributions are welcome but require explicit authorization from the author. Please submit a request via GitHub issues (https://github.com/michaelhuo/engine.bible/issues). The author reserves the right to deny contributions from entities whose beliefs conflict with orthodox Christian doctrine.
Contact
For authorization requests or support, open an issue at https://github.com/michaelhuo/engine.bible/issues or contact the author directly (contact details in LICENSE).
Acknowledgements

The Lockman Foundation for NASB authorization.
bible.world MediaWiki for wiki content integration.
FastAPI and SQLAlchemy communities for robust tools.
