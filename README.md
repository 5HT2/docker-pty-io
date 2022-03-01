# docker-pty-io

This is a command-line utility for interacting with Docker containers as terminals directly, essentially a direct STDIO interface. For example, if you have an SQL client opened in your docker container, you could run an SQL command and get the result without ever needing to SSH or do anything to open the container yourself.

## Building

A Makefile will be added shortly. For now, you can compile with the following commands.

```bash
git clone https://github.com/l1ving/docker-pty-io
cd docker-pty-io

# Build shim
cc -Wall -std=c99 shim.c -o shim
# Build cmd interface
go build -o cmd .
```

## Usage

Ensure that your container is running before the steps mentioned here.

Your container must be started in interactive mode for this to work (`-it`), as a limitation of `docker attach`. 
You can use `-d -it` to start it in detached like usual.

```bash
PORT="6016"
CONTAINER="my_container"

# Run PTY shim, needed to interface with docker attach.
# This is only required to be ran once per container.
# Each container should have it's own port assigned.
./shim $PORT "$CONTAINER"

# Give command to the container
./cmd $PORT "SELECT c1, c2 FROM t;"
```

## What? Why? Where?

- What is this actually doing?

  The `shim` program keeps a PTY <sup>[[def]](https://en.wikipedia.org/wiki/Pseudoterminal)</sup> opened on a local TCP server. This is needed to use `docker attach` without directly accessing it, as `docker attach` does not provide an easy way to detach the TTY (they ask you to Ctrl+C to close the program or to provide a detach sequence).
  The `cmd` program actually interfaces with the local PTY to give it STDIO and receive STDOUT as needed.
- Why would I use this? What is the advantage over [using an SQL client / other]?

  I did not intend for this to be a direct SQL interface, and it is quite versatile. The SQL examples are only there to get the concept across with a more familiar software for people.
- Where else is this used?

  I currently use this to send commands to a VS Server running on [`vintagestory-docker`](https://github.com/l1ving/vintagestory-docker/). This use-case makes much more sense as VS requires you keep a TTY opened to send commands to it, and this is not ideal with Docker containers.

## TODO

- [ ] Makefile
- [ ] "Interactive mode" (continuous commands rather than returning just once)
- [ ] More config options (maybe allow ./cmd "$container_name" or something)

## License

This project is licensed under [ISC](https://github.com/l1ving/docker-pty-io/blob/master/LICENSE.md) and applies to all files in the repository with the exception of `shim.c`.

The `shim.c` file is taken from [`iximiuz/ptyme`](https://github.com/iximiuz/ptyme) as a basic PTY server example, and is lightly modified to fit the needs of this project. It is not under a license, and should the author request I use something else I am more than happy to do so.
