This is a fork  [sarah](https://github.com/oklahomer/go-sarah) ```Adapter``` implementation for XMPP / Jabber

At present this is work in progress, and API may change without notice.


Upgrade for go-sarah v4 and go 1.17

# Getting Started
Below is a minimal sample that describes how to setup and start XMPP Adapter.


```go
package main

import (
	"fmt"
	"github.com/omacbr/go-sarah-xmpp"
	"io/ioutil"
	"os"
	"os/signal"
	"syscall"

	"github.com/oklahomer/go-kasumi/logger"
	"github.com/oklahomer/go-sarah/v4"
	"golang.org/x/net/context"
	"gopkg.in/yaml.v2"
)

func main() {
	// Setup configuration
	configBuf, err := ioutil.ReadFile("config.yaml")

	if err != nil {
		fmt.Println(err)
	}

	xmppConfig := xmpp.NewConfig()

	yaml.Unmarshal(configBuf, &xmppConfig)

	// Setup bot
	xmppAdapter, _ := xmpp.NewAdapter(xmppConfig)
	storage := sarah.NewUserContextStorage(sarah.NewCacheConfig())

	xmppBot := sarah.NewBot(xmppAdapter, sarah.BotWithStorage(storage))


	sarah.RegisterBot(xmppBot)

	// Start
	rootCtx := context.Background()
	runnerCtx, _ := context.WithCancel(rootCtx)

	err = sarah.Run(runnerCtx, sarah.NewConfig())
	if err != nil {
		panic(err)
	}

	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	signal.Notify(c, syscall.SIGTERM)

	select {
	case <-c:
		logger.Info("Stopping due to signal reception.")
		os.Exit(0)

	}

}
}
```

## Acknowledgements and thanks
This library uses the excellent xmpp library - https://github.com/mattn/go-xmpp
