# SteamUser
### A handler module for node-steam v1.0.0 and greater

SteamUser is a handler module for [node-steam](https://github.com/seishun/node-steam) version 1.0.0 or greater.

It's designed to be a self-contained module which provides all the functionality expected of a Steam user client.

**This is currently a work in progress and is incomplete.**

# Options

There are a number of options which can control the behavior of the `SteamUser` object. They are:

- `dataDirectory` - Controls where the Steam server list and sentry files are written. Defaults to a platform-specific user data directory.
	- On Windows, this is %localappdata%\doctormckay\node-steamuser
	- On Linux, this is ~/.local/share/node-steamuser
- `autoRelogin` - A boolean which controls whether or not `SteamUser` will automatically reconnect to Steam if disconnected for any reason (except an explicit logoff). Defaults to `true`.
- `singleSentryfile` - A boolean which controls whether or not `SteamUser` will use a single sentry file for all accounts. If off, a file named `sentry.accountname.bin` will be saved for each account. If on, a file named `sentry.bin` will be used for all accounts. Defaults to `false`.
- `promptSteamGuardCode` - A boolean which controls whether or not `SteamUser` will automatically prompt for Steam Guard codes when necessary from `stdin`.

# Properties

### client

The `SteamClient` which is being used to communicate with Steam.

### steamID

`null` if not connected, a [`SteamID`](https://www.npmjs.com/package/steamid) containing your SteamID otherwise.

### options

An object containing options for this `SteamUser`. **Read-only**, use `setOption` or `setOptions` to change an option.

### emailInfo

An object containing information about your account's email address. `null` until [`emailInfo`](#emailinfo-1) is emitted.

- `address` - Your email address
- `validated` - `true` if your email is validated, `false` if not

### limitations

An object containing information about your account's limitations. `null` until [`accountLimitations`](#accountlimitations) is emitted.

- `limited` - `true` if your account is [limited](https://support.steampowered.com/kb_article.php?ref=3330-IAGK-7663), `false` if not
- `communityBanned` - `true` if your account is banned from Steam Community, `false` if not
- `locked` - `true` if your account is [locked](https://support.steampowered.com/kb_article.php?ref=6416-FHVM-3982), `false` if not (accounts can also be locked by Support)
- `canInviteFriends` - `true` if your account can invite friends, `false` if not

### wallet

An object containing information about your Steam Wallet. `null` until [`wallet`](#wallet-1) is emitted.

- `hasWallet` - `true` if your account has a Steam Wallet, `false` if not
- `currency` - The ID of your wallet's currency
- `balance` - Your account's wallet balance, in the lowest currency denomination (e.g. USD cents)

### licenses

An array containing license data for the packages which your Steam account owns. `null` until [`licenses`](#licenses-1) is emitted.

# Methods

### Constructor([client][, options])
- `client` - An optional `SteamClient` to use to connect to Steam. If not provided, one will be created automatically.
- `options` - An optional object containing zero or more [options](#options) to set for this `SteamUser`.

Constructs a new `SteamUser`.

### setOption(option, value)
- `option` - The name of the option to set
- `value` - The value to set for this option

Changes the value of an [option](#options).

### setOptions(options)
- `options` - An object containing zero or more [options](#options).

### logOn([details])
- `details` - An object containing details for this logon
	- `accountName` - If logging into a user account, the account's name
	- `password` - If logging into an account without a login key, the account's password
	- `loginKey` - If logging into an account with a login key, this is the account's login key
	- `rememberPassword` - `true` if you want to get a login key which can be used in lieu of a password for subsequent logins. `false` or omitted otherwise.

Logs onto Steam. Omit the `details` object if you wish to login to an anonymous user account.

### logOff()

Logs you off of Steam and closes the connection.

### webLogOn()

`SteamUser` will automatically log onto steamcommunity.com when a successful connection to Steam is established (as an individual user), but you can call `webLogOn()` to create a new session if your old one expires or becomes invalid.

Listen for the [`webSession`](#websession) event to get your cookies.

### gamesPlayed(apps)
`apps` - An array, object, string, or number (see below)

Reports to Steam that you're playing or using zero or more games/apps. To exit all games/apps, use an empty array `[]`.

To play a single game by AppID, use a single integer (e.g. `440`)

To play a single non-Steam game by name, use a single string (e.g. `"Minecraft"`)

To play a single game by AppID and name (the client-provided name is what is given to the WebAPI and mobile app), use an object of this format:

```js
{
	"game_id": 440,
	"game_extra_info": "Team Fortress 2"
}
```

You can use multiple apps by providing an array of any mixture of the above formats.

### getPlayerCount(appid, callback)
- `appid` - The AppID of the app for which you'd like the current player/user count (use `0` to get current logged-in Steam user count)
- `callback` - Called when the requested data is available
	- `result` - A value from `Steam.EResult`
	- `players` - How many Steam users are currently playing/using the app

Requests a count of how many Steam users are currently playing/using an app.

### getServerList(filter, limit, callback)
- `filter` - A master server [filter string](https://developer.valvesoftware.com/wiki/Master_Server_Query_Protocol#Filter)
- `limit` - How many servers should be returned, at maximum. Hard limit is 5000.
- `callback` - Called when the requested data is available
	- `servers` - An array of objects containing server data
		- `addr` - The server's IP address in `x.x.x.x:p` format
		- `gameport` - The port the server is running on for game clients
		- `specport` - The port the server is running on for spectator clients (`null` for none)
		- `steamid` - A [`SteamID`](https://www.npmjs.com/package/steamid) object containing the server's SteamID
		- `name` - The server's hostname
		- `appid` - The AppID of the game which the server is serving
		- `gamedir` - The directory of the game which the server is serving
		- `version` - The version of the game which the server is serving
		- `product` - The product name of the game which the server is serving
		- `region` - The region code for where the server is located
		- `players` - How many people are currently on this server
		- `max_players` - How many people can be on the server at once
		- `bots` - How many CPU players are currently on this server
		- `map` - The name of the map which the server is currently running
		- `secure` - `true` if the server is VAC-secure, `false` if not
		- `dedicated` - `true` if the server is dedicated, `false` if listen
		- `os` - `w` if the server is running on Windows, `l` for Linux
		- `gametype` - The server's tags, separated by commas

Requests a list gameservers from Steam matching a given filter, along with information about the server as Steam knows it.

### getProductChanges(sinceChangenumber, callback)
- `sinceChangenumber` - The changenumber of the last known changelist. You will get changes which have occurred since then and now. Use 1 to request all changes ever.
- `callback` - Called when data is available
	- `currentChangenumber` - The changenumber of the newest changelist
	- `apps` - An array of objects for apps which have changed. Each object has these properties:
		- `appid` - The AppID of the app
		- `change_number` - The changenumber of the latest changelist in which the app has changed
		- `needs_token` - `true` if you need an authorization token to get most details about this app, `null` if not
	- `packages` - An array of objects for packages which have changed. Each object has the same properties as the `apps` array, except `appid` is `packageid`.

Requests a list of all apps/packages which have changed since a given changenumber.

# Events

### loggedOn
- `details` - An object containing various details about your account (see [`CMsgClientLogonResponse`](https://github.com/SteamRE/SteamKit/blob/SteamKit_1.6.3/Resources/Protobufs/steamclient/steammessages_clientserver.proto#L93-L116))

Emitted when you're successfully logged into Steam.

### steamGuard
- `domain` - If an email code is needed, the domain name of the address where the email was sent. `null` if an app code is needed.
- `callback` - Should be called when the code is available.
	- `code` - The Steam Guard auth code

If the `promptSteamGuardCode` option is disabled, this event will be emitted when Steam requests a Steam Guard code from us. You should collect the code from the user somehow and then call the `callback` with the code as the sole argument.

Example:

```js
user.on('steamGuard', function(domain, callback) {
	console.log("Steam Guard code needed from email ending in " + domain);
	var code = getCodeSomehow();
	callback(code);
});
```

### error
- `e` - An `Error` object

Emitted when an error occurs during logon. If this event isn't handled, the program will crash.

The `Error` object will have an `eresult` parameter which is a value from the [`EResult`](https://github.com/SteamRE/SteamKit/blob/SteamKit_1.6.3/Resources/SteamLanguage/eresult.steamd) enum.

### webSession
- `sessionID` - The value of the `sessionid` cookie
- `cookies` - An array of cookies, as `name=value` strings

Emitted when a steamcommunity.com web session is successfully negotiated. This will automatically be emitted on logon and in response to [`webLogOn`](#weblogon) calls.

Some libraries require you to provide your `sessionID`, others don't. If your library doesn't, you can safely ignore it.

### loginKey
- `key` - Your login key

If you enabled `rememberPassword` in [`logOn`](#logondetails), this will be emitted when Steam sends us a new login key. This key can be passed to [`logOn`](#logondetails) as `loginKey` in lieu of a password on subsequent logins.

At this time, I'm not sure if login keys expire, so to be safe you should record this somewhere (in a database, in a file, etc) and overwrite it every time the event is emitted.

### newItems
- `count` - How many new items you have (can be 0)

Emitted when Steam sends a notification of new items. You can reset the count to 0 by loading your inventory page (https://steamcommunity.com/my/inventory) while logged in.

### newComments
- `count` - How many total new comments you have (can be 0)
- `myItems` - How many of the total comments are on your content (workshop items, screenshots, your profile, etc.)
- `discussions` - How many of the total comments are posts in subscribed discussion threads

Emitted when Steam sends a notification of new comments.

### tradeOffers
- `count` - How many active received trade offers you have (can be 0)

Emitted when Steam sends a notification of new trade offers.

### offlineMessages
- `count` - How many unread offline chat messages you have
- `friends` - An array of SteamID strings for the users who have sent you unread offline chat messages

Emitted when Steam sends a notification of unread offline chat messages. This will always be emitted after logon, even if you have no messages.

### emailInfo
- `address` - Your account's email address
- `validated` - A boolean value for whether or not your email address is validated

Emitted on logon and when email info changes. The [`emailInfo`](#emailinfo) property will be updated after this event is emitted.

### accountLimitations
- `limited` - `true` if your account is [limited](https://support.steampowered.com/kb_article.php?ref=3330-IAGK-7663), `false` if not
- `communityBanned` - `true` if your account is banned from Steam Community, `false` if not
- `locked` - `true` if your account is [locked](https://support.steampowered.com/kb_article.php?ref=6416-FHVM-3982), `false` if not (accounts can also be locked by Support)
- `canInviteFriends` - `true` if your account can invite friends, `false` if not

Emitted on logon and probably when limitations change. The [`limitations`](#limitations) property will be updated after this event is emitted.

### wallet
- `hasWallet` - `true` if your account has a Steam Wallet, `false` if not
- `currency` - The currency ID of your account's wallet
- `balance` - Your account's current wallet balance in the lowest currency denomination (e.g. USD cents)

Emitted on logon and when wallet balance changes. The [`wallet`](#wallet) property will be updated after this event is emitted.

### licenses
- `licenses` - An array of licenses

Contains the license data for the packages which your Steam account owns. To see license object structure, see [`CMsgClientLicenseList.License`](https://github.com/SteamRE/SteamKit/blob/SteamKit_1.6.3/Resources/Protobufs/steamclient/steammessages_clientserver.proto#L307-L320).

Emitted on logon and when licenses change. The [`licenses`](#licenses) property will be updated after this event is emitted.