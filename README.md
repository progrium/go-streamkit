# Streamkit

Streamkit is an application communications library built on top of the SSH protocol stack. It gives you secure pipes over TCP and between processes that tunnel bi-directional data streams or message frames. It also works efficiently in-process.

This project is still in proof of concept phase. You can see an example of it in the `example` directory, but also [localtunnel](https://github.com/progrium/localtunnel) was rewritten to use Streamkit (when it was called Duplex).

## API

The API has gone through many changes but this is currently what it looks like in rough form, written as Go interfaces.

	type Peer interface {
		// Options
		SetOption(option int, value interface{}) error
		GetOption(option int) interface{}

		// Connections
		Connect(endpoint string) error
		Disconnect(endpoint string) error
		Bind(endpoint string) error
		Unbind(endpoint string) error

		// Remote Peers
		Peers() []string
		Drop(peer string) error
		NextPeer() string

		// Channels
		Accept() (ChannelMeta, Channel)
		Open(peer, service string, headers []string) (Channel, error)

		// Cleanup
		Shutdown() error
	}

	type ChannelMeta interface {
		// Name of service, if any
		Service() string

		// Headers, often key=vaue
		Headers() []string

		// Trailers, or "close headers"
		// Available once closed.
		Trailers() []string

		// Name of peers
		LocalPeer() string
		RemotePeer() string
	}

	type Channel interface {
		// Send and receive
		Write(data []byte) (int, error)
		Read(data []byte) (int, error)

		// Frames
		WriteFrame(frame []byte) error
		ReadFrame() ([]byte, error)

		// Errors
		WriteError(frame []byte) error
		ReadError() ([]byte, error)

		// EOF, Trailers, Close
		CloseWrite() error
		WriteTrailers(trailers []string) error
		Close() error

		// Channels of Channels
		Open(service string, headers []string) (Channel, error)
		Accept() (ChannelMeta, Channel)

		// Attach to real sockets for gateways/proxies
		Join(rwc io.ReadWriteCloser)

		// Reference to ChannelMeta
		Meta() ChannelMeta
	}


## Architecture

#### Peers and Channels

The main primitives in Streamkit are Peers and Channels. Peers are like advanced sockets that can connect and be connected to. They most closely resemble ZeroMQ sockets. Behind the scenes they are both an SSH server and client up to the SSH Connection Layer. There's no terminals or shelling out here, we just use the lower level protocols that give us encrypted, bi-directional, multiplexed streams. These streams are exposed as Channels.

Channels are much closer to a regular socket connection, but are tunneled through the secure connections between Peers. Unlike regular TCP connections, Channels come with some metadata, including their intended service and key-value headers. The Channel API has basic send/recv calls, but also calls for sending and receiving message frames. It also utilizes what SSH would use for stderr to send error frames.

Frames are just length prefixed payloads of bytes. Frames can go in either direction. This is a solid foundation for any application protocol. What's more, you can send Channels over Channels. Just think about that.


## License

MIT
