# Kiwix Server — Docker Compose Setup

> *This README is a companion to the YouTube video walkthrough — follow along or use this as a standalone reference.*

---

## Why This Matters

Access to information is a fundamental human right.

Information is the greatest tool any person can possess — it shapes how we understand the world, make decisions, protect ourselves, and grow. Yet across the globe, access to that information is increasingly filtered, paywalled, censored, or simply unavailable due to politics, infrastructure or conflict.

No government, corporation, or institution has the right to bar people from knowledge. The truth belongs to no one — and therefore belongs to everyone. People deserve the ability to seek it out, weigh it themselves, and come to their own conclusions. That process only works when access is free and open.

Kiwix exists in that spirit. It lets anyone host and serve entire libraries of human knowledge — offline, independently, and without anyone's permission. Wikipedia. Medical references. Educational textbooks. Historical archives. All of it, on your own hardware, available to anyone you choose to share it with.

This guide will help you run your own Kiwix server. Own your information. Share it freely.

---

[Kiwix](https://www.kiwix.org) lets you host offline copies of websites (Wikipedia, Stack Overflow, Project Gutenberg, etc.) served as `.zim` files. This setup runs Kiwix via Docker Compose.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed (included with Docker Desktop)
- At least one `.zim` file downloaded (see [Downloading ZIM Files](#downloading-zim-files))

---

## Directory Structure

```
kiwix/
├── docker-compose.yml
└── data/              # Place your .zim files here
    └── wikipedia_en_all_maxi.zim
```

---

## Configuration

### `docker-compose.yml`

```yaml
services:
  kiwix:
    image: ghcr.io/kiwix/kiwix-serve:latest
    container_name: kiwix
    restart: unless-stopped
    ports:
      - "8082:8080"
    volumes:
      - /PATH:/data
    command:
      - '*.zim'
```

**Key settings:**

| Setting | Description |
|---|---|
| `ports: "8082:8080"` | Exposes the server on host port `8082`. Change the left value to use a different port. |
| `volumes: /PATH:/data` | Replace `/PATH` with the absolute path to your folder containing `.zim` files. |
| `command: '*.zim'` | Serves all `.zim` files found in `/data`. |
| `restart: unless-stopped` | Automatically restarts the container on reboot or crash. |

### Example volume path

Replace `/PATH` with your actual directory:

```yaml
volumes:
  - /home/user/kiwix/data:/data
  # or on macOS:
  - /Users/user/kiwix/data:/data
```

---

## Downloading ZIM Files

Browse and download `.zim` files from the official Kiwix library:

**[https://library.kiwix.org](https://library.kiwix.org)**

You can also use the `kiwix-manage` CLI or download directly with `wget`/`curl`. Example:

```bash
# Download a small ZIM for testing (Stack Overflow in English, mini)
wget https://download.kiwix.org/zim/stack_exchange/stackoverflow.com_en_mini_2024-05.zim -P ./data
```

Place all downloaded `.zim` files into the directory mapped to `/data` in the volume.

---

## Starting the Server

```bash
# Start in the background
docker compose up -d

# View logs
docker compose logs -f kiwix

# Stop the server
docker compose down
```

---

## Accessing the Server

Once running, open your browser and navigate to:

```
http://localhost:8082
```

From another device on your local network, replace `localhost` with the host machine's IP address:

```
http://192.168.1.x:8082
```

---

## Updating

To pull the latest Kiwix image:

```bash
docker compose pull
docker compose up -d
```

---

## Troubleshooting

**No books appear after startup**
- Confirm `.zim` files exist in the directory mapped to `/data`.
- Check that the path in `volumes` is an absolute path, not relative.
- Run `docker compose logs kiwix` to see what files were detected.

**Port already in use**
- Change the host port in `docker-compose.yml` (left side of `ports`), e.g. `"8083:8080"`.

**Container exits immediately**
- Ensure at least one `.zim` file is present — the server requires a file to serve on startup.
- Verify the volume path exists and is readable by Docker.
