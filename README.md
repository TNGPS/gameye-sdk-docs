# Gameye SDK Documentation

## intro

Create eSport and competitive matches for for your platform 
without fixed monthly costs or any need for your own server infrastructure for games like:
- Counter-Strike: Global Offensive, 
- Team Fortress 2, Left 4 Dead 2, Killing Floor 2, Insurgency and Day of Infamy 

Simply implement the Gameye API to kick off online matches when you need the
you will even be able to implement the scores/statistics directly on your website. 

How cool is that!

# availabkle SDK options
The Gameye SDK's are the recommended way to use out API's.

SDK are available in several popular languages:
1. Typescript (node.js)
1. Golang
1. PHP
1. raw REST API (curl)
1. request more languages ....

In these pages we well show you how easy it is 
to use the SDK's and tge raw REST API's.

## Get started

### get an API KEY

Obtain a free Gameye API key, please send us an [email](mailto:support@gameye.com)

All API request need to be authorised using your `API_KEY` 
as an (oauth) `Bearer` token

### install the SDK
Follow the in instructions for your language of choice
bellow. In case of any trouble, check out out FAG or 
or contact support, for assistance

#### install the SDK (node.js using npm)

```bash
$ npm install @gameye/sdk -s
```

#### install the SDK (Golang)
```goLang
TODO ....
```

#### install the SDK (PHP using composer)

```php
$ composer require gameye/gameye-sdk-php
{
    "require": {
        "gameye/gameye-sdk-php": "2.*"
    }
}
```

# Create Gameye client

The SDK's provide a Gameye client class, you will need to instantiate it with your `GAMEYE_API_KEY` 
and the API `endoint` you want to use. 

In case of raw API use, you just addres the correct `endpoint` and add the `GAMEYE_API_KEY` in the
authorization header.

## Instantiate a Gameye Client (typescript)

```typescript
import { GameyeClient, GameyeClientConfig } from '@gameye/sdk'

const api_config = <GameyeClientConfig>({endpoint:'https://api.gameye.com', token: 'GAMEYE_API_KEY'});

const gameye = new GameyeClient(api_config);

```

## Instantiate a Gameye Client (PHP)

```PHP
$gameye = new \Gameye\SDK\GameyeClient([
            'AccessToken' => 'GAMEYE_API_KEY',
            'ApiEndpoint' => 'https://api.gameye.com',
        ]);
```

# Command and Query Responsibility Segregation (CQRS)

The Gameye SDK's (and API) are desigend using the
[Command and Query Responsibility Segregation](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/)
design pattern.

We have disticts set or `queries` you can use the GET information from the API servers, 
and we have an other set of `commands` you can user to change things. You will use the 
`query` interface to get information about possible games and options and about statistics of
a match in progress. However you will need to use the `command` interface for things like `
`start`ing and `stop`ping a match etc.


## Get information about available games and locations

A wide range of games and game opions are available, 
you can `GET` an up to date list using the `game` `Query`.

The result will be a structure with a list of `game`s, a list of `location`s and 
for each `game` a list of locations where the game is available.

###  Query Game Sample Data (json)

The game and location data are represented as a simple json structure like this:

```json
{
    "game": {
        "csgo": {
            "gameKey": "csgo",
            "location": {
                "rotterdam": true,
                "los_angeles": true,
                "washington_dc": true,
                "frankfurt": true
            }
        },
        "tf2": {
            "gameKey": "tf2",
            "location": {
                "rotterdam": true
            }
        },
        "css": {
            "gameKey": "css",
            "location": {}
        }
    },
    "location": {
        "rotterdam": {
            "locationKey": "rotterdam"
        },
        "dubai": {
            "locationKey": "dubai"
        },
        "tokyo": {
            "locationKey": "tokyo"
        },
        "los_angeles": {
            "locationKey": "los_angeles"
        }
    }
}
```

###  Query Game (raw API)

Note that the basic information about games and locations is public and you can fetch this data
without the need for an valid `GAMEYE_API_KEY`

```bash
curl https://api.gameye.com/fetch/game
```


###  Query Game (typescript)

In `node.js` (typescipt) the `query` interface returns a `promise`. 

You can use the clasic `.then().catch()` syntax like this:

```typescript
gameye.queryGame().then(games_and_locations => {
    // extract game and location data
    console.debug("Games and locations available:");
    for(const game in games_and_locations.game){
        console.log('game : ', game, ' available at locations:', games_and_locations.game[game].location);
    }
    for(const location in games_and_locations.location){
        console.log('location : ', location,  games_and_locations.location[location]);
    }
}).catch(error=>{
    console.error("Sorry: queryGame call failed: ", error);
});

```

However we prever to use the `async`, `await` [syntax](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-7.html).
like this:

```typescript
async function get_games_and_locations(gameye: GameyeClient) {

    console.log("\nGet available games and locations ...");
    let games_and_locations;
    try {
        games_and_locations = await gameye.queryGame();
    } catch (error) {
        console.error("Sorry: queryGame call failed: ", error);
        return -1;
    }
    console.log("... done");

    for (const game in games_and_locations.game) {
        console.log("game : ", game, " available at locations:", games_and_locations.game[game].location);
    }

    for (const location in games_and_locations.location) {
        console.log("location : ", location, "  -->", games_and_locations.location[location]);
    }
}
```

###  Query Game (Golang)

TODO

###  Query Game (PHP)

TODO

## Query Game templates and parameters

For each game a variety of templates exists to start a match. The API allows you to get 
an up to date list of known templates and the allowed argument (value)s.

 
### Template Sample Data (json)

Fetching the list of templates for a valid `gameKey` results in a json structure where
all valid templates are listed as keys in the `template` attribute.

Each `template` has a `templateKey` and an `arg` attribute.

The `arg` attribute list all the valid arguments than can be passed when staring a match.

Arguments have:
 - a `name`, 
 - a `type` (`string` or `number`) 
 - a `defaultValue`
 - a `minimumValue` and `maximumValue` in case of numeric values
 - an `option` array of allowed values of the correct type
 - a `validatePattern` (string) and a `validateIgnoreCase` (boolean) for `string` 
   arguments that need to be validated against a regex.
 
A sample looks like this:
 
```json
{
	"template": {
		"2v2wingman": {
			"templateKey": "2v2wingman",
			"arg": []
		},
		"bots": {
			"templateKey": "bots",
			"arg": [{
				"name": "tickRate",
				"type": "number",
				"defaultValue": 128,
				"option": [64, 128]
			}, {
				"name": "steamToken",
				"type": "string",
				"defaultValue": ""
			}, {
				"name": "hostname",
				"type": "string",
				"defaultValue": "gameye.com Match Server"
			}, {
				"name": "mapgroup",
				"type": "string",
				"defaultValue": "mg_active",
				"validatePattern": "^\\w+$",
				"validateIgnoreCase": true
			}, {
				"name": "map",
				"type": "string",
				"defaultValue": "de_dust2",
				"validatePattern": "^\\w+$",
				"validateIgnoreCase": true
			}, {
				"name": "gameMode",
				"type": "number",
				"defaultValue": 1,
				"option": [0, 1, 2]
			}, {
				"name": "gameType",
				"type": "number",
				"defaultValue": 0,
				"option": [0, 1]
			}, {
				"name": "maxRounds",
				"type": "number",
				"defaultValue": 30,
				"minimumValue": 1,
				"maximumValue": 30
			}]
		},
		"esl1on1": {
			"templateKey": "esl1on1",
			"arg": []
		}
	}
}
```


### Query Template (raw API)

```bash
curl \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_KEY" \
https://api.gameye.com/fetch/template/VALID_GAME_KEY
```

### Query Template (node, typescript)

```typescript
async function get_templates_for_game(gameye: GameyeClient, gameKey: string) {

    let available_templates: any;

    console.log("\nGet avaiable templates for game", gameKey, " ...");
    try {
        available_templates = await gameye.queryTemplate(gameKey);
    } catch (error) {
        console.error("Sorry: queryTemplate call failed: ", error);
        return -1;   
    }
    console.log("...done");
    
    console.log('\nknown templates for game: ', gameKey);

    for (const template in available_templates.template) {
        console.log('known template for game: ', gameKey,  ' --> ', template, ' --> ',  available_templates.template[template]);
    }
}
```



## Start a match

At the command site of the API we have two command, one to `start` a match an one for `stop`ing the match.

`Command`s will not give a result back (just a status code indicating success or error), 
therefore you need to provide your own unique `matchKey` as a reference for the match.

In the sample code we a timestamp with suffient large resolution as a match key, 
but any unique (string) ID will do.

To `start` a match you need the following input:
1. a `gameKey` to specify the game (e.g. 'csgo')
1. a list of `locationKeys` where you want your servers to be deployed
1. a `templateKey` to select a known game template (see above)
1. a unique `matchkey` for reference of the match
1. optional a set of match `config`uration parameters (see above)


### start and stop a match (raw API)

In case of the raw API you simply POST all argumments 
to the `/action/start-match` like this:

```bash
curl \
--request POST \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_KEY" \
--data '{
    "locationKeys": ["rotterdam"],
    "gameKey": "csgo",
    "templateKey": "bots",
    "matchKey": "YOUR_UNIQUE_MATCH_KEY",
    "config": {
      "steamToken": "YOUR_STEAM_TOKEN",
      "maxPlayers": 12,
      "tickRate":  128,
      "maxRounds": 6,
      "gameType": 0,
      "gameMode": 1,
      "mapgroup": "mg_active",
      "map": "de_dust",
      "teamNameOne": "TeamA",
      "teamNameTwo": "TeamB"
   }
}' \
https://api.gameye.com/action/start-match
```

and to `stop` use the `/action/stop-match` end point

```bash
curl \
--request POST \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_KEY" \
--data '{
    "matchKey": "YOUR_UNIQUE_MATCH_KEY"
}' \
https://api.gameye.com/action/stop-match
```

The result wil be a HTTP status code:
- a `204` 'NO CONTENT' in case of a successful started match
- a `200` 'SUCCESS' in case of a successful stopped match
- a `401` 'AUTHORISATION NEEDED' in case of a missing `GAMEYE_API_KEY` 
- a `403` 'NOT ALLOWED' in case of an invalid `GAMEYE_API_KEY` 
- a `404` 'NOT FOUND' in case of a non existing game (template)
- a `500` in case of any mistakes in the posted payload
 
 

### start and stop a match (node, typescript)

```typescript
async function start_stop_match(gameye: GameyeClient){
    const gameKey: string = "csgo";
    const locationKeys: string[] = ["rotterdam"];
    const templateKey: string = "bots";
    const matchConfig = {
     "steamToken": "",
     "maxPlayers": 12,
     "tickRate":  128,
     "maxRounds": 6,
     "gameType": 0,
     "gameMode": 1,
     "mapgroup": "mg_active",
     "map": "de_dust",
     "teamNameOne": "TeamA",
     "teamNameTwo": "TeamB"
   };
    const matchKey: string = '' + new Date().getTime(); // we use a timestamp as a unique gameKey

    console.log("\nlet's START a match with key", matchKey , " ...");
    try {
        await gameye.commandStartMatch(matchKey, gameKey, locationKeys, templateKey, matchConfig);
    } catch (error) {
        console.error("Sorry: commandStartMatch call failed: ", error);
        return -1;
    }
    console.log("...done");
    
    // start handling match updates here ...
   
    console.log("\nlet's STOP the game ...");
    try{
        await gameye.commandStopMatch(matchKey);
    } catch (error) {
        console.error("Sorry: commandStartMatch call failed: ", error);
        return -1;        
    }
    console.log("...done");

}
```

## Listen to real time updates of a running match

After you stared a match you can listen to updates from the live match using 
a view simple `query` calls.

If you have any matches in progress you can fetch the state using
the  `match` `query`.

### Match data sample (json)
The match query list all matches indexed by `matchKey` 
```json
{ 
    match: { 
     '1536150262360': null,
     '1536150484784': null,
     '1536150547618': {
        matchKey: '1536150547618',
        gameKey: 'csgo',
        locationKey: 'rotterdam',
        host: '213.163.71.5',
        created: 1536150547865,
        port: { 
          game: 50716, 
          gotv: 54478
        }
     }
}
```

### Query match state / statistics (raw API)

```bash
curl \
--header "Authorization: Bearer GAMEYE_API_KEY" \
https://api.gameye.com/fetch/match/VALID_MATCH_KEY
```

### Query match state / statistics (node,js, typescript)

```typescript

async function request_match(gameye: GameyeClient, matchKey: string){

    // get all info about running matches
    let all_matches: any;
    try {
        all_matches = await gameye.queryMatch(); // TODO: suggest to pass matchKey for this query also
        console.log('  - my match = ', all_matches.match[matchKey]);
    } catch (error) {
        console.warn('Problem with queryMatch :', error);
    }
}
```
    
