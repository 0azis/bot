# Golang Telegram Bot

> The project is under development. API may be changed before v1.0.0 version.

> [Telegram Group](https://t.me/gotelegrambotui)

It's a Go zero-dependencies telegram bot framework

A simple example `echo-bot`:

```go
package main

import (
	"context"
	"os"
	"os/signal"
	"strconv"

	"github.com/go-telegram/bot"
	"github.com/go-telegram/bot/methods"
	"github.com/go-telegram/bot/models"
)

// Send any text message to the bot after the bot has been started

func main() {
	opts := []bot.Option{
		bot.WithDefaultHandler(handler),
	}

	b := bot.New("YOUR_BOT_TOKEN_FROM_BOTFATHER", opts...)

	ctx, cancel := signal.NotifyContext(context.Background(), os.Interrupt)
	defer cancel()

	b.Start(ctx)
}

func handler(ctx context.Context, b *bot.Bot, update *models.Update) {
	methods.SendMessage(ctx, b, &methods.SendMessageParams{
		ChatID: update.Message.Chat.ID,
		Text:   update.Message.Text,
	})
}
```

You can find more examples in the [examples](examples) folder.

For test examples, you should set environment variable `EXAMPLE_TELEGRAM_BOT_TOKEN` to your bot token.

## Getting started

Go version: **1.18**

Install the dependencies:

```bash
go get -u github.com/go-telegram/bot
```

Initialize and run the bot:

```go
b := bot.New("YOUR_BOT_TOKEN_FROM_BOTFATHER")

b.Start(context.TODO())
```

You can to define default handler for the bot:

```go
b := bot.New("YOUR_BOT_TOKEN_FROM_BOTFATHER", bot.WithDefaultHandler(handler))

func handler(ctx context.Context, b *bot.Bot, update *models.Update) {
	// this handler will be called for all updates
}
```

## Available methods

All available methods are listed in the [Telegram Bot API documentation](https://core.telegram.org/bots/api)

You can find these methods in the `methods` package. All methods have name like in official documentation, but with capital first letter.

`methods.SendMessage`, `methods.GetMe`, `methods.SendPhoto`, etc

## Options

You can use options to customize the bot.

```go
b := bot.New("YOUR_BOT_TOKEN_FROM_BOTFATHER", opts...)
```

Full list of options you can find [here](options.go)

## Message.Text and CallbackQuery.Data handlers

For your convenience, you can use `Message.Text` and `CallbackQuery.Data` handlers.

An example:

```go
b := bot.New("YOUR_BOT_TOKEN_FROM_BOTFATHER")

b.RegisterHandler(bot.HandlerTypeMessageText, "/start", bot.MatchTypeExact, myStartHandler)

b.Start(context.TODO())
```

> also you can use options `WithMessageTextHandler` and `WithCallbackQueryDataHandler`


In this example, the handler will be called when the user sends `/start` message. All other messages will be handled by the default handler.

Handler Types:
- `HandlerTypeMessageText` - for Update.Message.Text field
- `HandlerTypeCallbackQueryData` - for Update.CallbackQuery.Data field

RegisterHandler returns a handler ID string. You can use it to remove the handler later.

```
b.UnregisterHandler(handlerID)
```

Match Types:
- `MatchTypeExact` 
- `MatchTypePrefix` 
- `MatchTypeContains`

Also, you can use `RegisterHandlerRegexp` to match by regular expression.

```go
re := regexp.MustCompile(`^/start`)

b.RegisterHandlerRegexp(bot.HandlerTypeMessageText, re, myStartHandler)
```

## InputFile

For some methods, like `SendPhoto`, `SendAudio` etc, you can send file by file path or file contents.

For send file by URL or FileID, you can use `&models.InputFileString{Data: string}`:

```go
// file id of uploaded image 
inputFileData := "AgACAgIAAxkDAAIBOWJimnCJHQJiJ4P3aasQCPNyo6mlAALDuzEbcD0YSxzjB-vmkZ6BAQADAgADbQADJAQ"
// or URL image path
// inputFileData := "https://example.com/image.png"

params := &methods.SendPhotoParams{
    ChatID:  chatID,
    Photo:   &models.InputFileString{Data: inputFileData},
}

methods.SendPhoto(ctx, b, params)
```

[Demo in examples](examples/send_photo)

For send image file by file contents, you can use `&models.InputFileUpload{Filename: string, Data: io.Reader}`:

```go
fileContent, _ := os.ReadFile("/path/to/image.png")

params := &methods.SendPhotoParams{
    ChatID:  chatID,
    Photo:   &models.InputFileUpload{Filename: "image.png", Data: bytes.NewReader(fileContent)},
}

methods.SendPhoto(ctx, b, params)
```

[Demo in examples](examples/send_photo_upload)

## InputMedia

For methods like `SendMediaGroup` or `EditMessageMedia` you can send media by file path or file contents.

[Official documentation InputMedia](https://core.telegram.org/bots/api#inputmedia)

> field `media`: File to send. Pass a file_id to send a file that exists on the Telegram servers (recommended), pass an HTTP URL for Telegram to get a file from the Internet, or pass “attach://<file_attach_name>” to upload a new one using multipart/form-data under <file_attach_name> name.

If you want to use `attach://` format, you should to define `MediaAttachment` field with file content reader.

```go
fileConetent, _ := os.ReadFile("/path/to/image.png")

media1 := &models.InputMediaPhoto{
	Media: "https://telegram.org/img/t_logo.png",
}

media2 := &models.InputMediaPhoto{
	Media: "attach://image.png", 
	Caption: "2",
	MediaAttachment: bytes.NewReader(fileConetent),
}

params := &methods.SendMediaGroupParams{
    ChatID: update.Message.Chat.ID,
    Media: []models.InputMedia{
        media1,
        media2,
    },
}

methods.SendMediaGroup(ctx, b, params)
```

[Demo in examples](examples/send_media_group)

## UI Components

In the repo https://github.com/go-telegram/ui you can find a some UI elements for your bot.

- datepicker
- inline_keyboard
- slider
- paginator

and more...

Please, check the repo for more information and live demo.