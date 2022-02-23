# Notion game database

[![https://imgur.com/ubzY4G7.jpg](https://imgur.com/ubzY4G7.jpg)](https://imgur.com/ubzY4G7.jpg)

Since there are so many game launchers now a days and it's hard to keep track on what you own where. That is why I keep my game database in Notion. I was missing a lot images, so I connected the IGDB database to the Notion API. If you launch the site it will list all your games without an cover at the left most column and when you click a game it will load 5 images from the IGDB when you select one it will set it as the cover in Notion and load the next game. 

This is a simple website that gives you a UI to work with the two APIs, it's quickly setup with no fancy featues, but hopefully it will help someone.
## Project Setup
### Launch chrome with CORS disabled
```sh
open -n -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --user-data-dir="/tmp/chrome_dev_sess_1" --disable-web-security
```

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```


### `secrets.json`
Rename `secrets-example.json` and update with your own credentials
```json
{
  "notion": {
    "token": "NOTION_SECRET",
    "databaseId": "DATABASE_ID"
  },
  "igdb": {
    "clientID": "CLIENT_ID",
    "token": "TOKEN"
  }
}
``