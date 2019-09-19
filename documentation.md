*API documentation*

In order to implement provably fair in the extension, there is a set of data required from any operator. The standard questions for data retrieval are as per below:

1. get next server seed hash
2. submit / mutate client seed
3. script info for verification like how nonce are appended to compute outcome
4. get all bets for a user ( with primary key like id)
5. reveal unhashed server seed after the bets are placed
6. bet specific info like seeds used for that bet (with primary key like id)

Below are the operator wise details.


Crypto Games
Fairly everything available on  https://api.crypto-games.net/v1/

Authentication : user needs to pass their API key which can be obtained from operator


Bitvest
Authentication : 

User needs to be logged in and the session id cookie is used for authentication

Other headers passed are token & secret

Token is obtained by creating a half account in the beginning and secret is always used 0 for full accounts

for more details please check diceboot implementation
https://github.com/Seuntjie900/DiceBot/blob/master/DiceBot/Bitvest.cs


The result of every game should be equal to:
HMAC_SHA512(ServerSeed,PlayerSeed|Nonce)

nonce goes 0,1,2,3,...
if server seed changed nonce 0 
if client seed changed nonce 0
if server seed changes client seed needs to be changed!

You may request a new server seed any time to allow for verification of your previous bets.
The ServerHash should equal SHA256(ServerSeed).

Posting to update with self_only=1 will only give information specific to your account. 

Posting on various endpoints will reveal data / perform action as below

Action.php endpoint

1) act : play_dice

data passed

side: low
target: 49.9999
token: 4ug2IKQd0tXPRX
secret: 0
bet: 0
user_seed: e28911518b445760c36ff68f126c4d4a028f7d7633a80b3a25650d2f66ead96c
currency: bitcoins
act: play_dice
v: 101

response

data: {balance: "0.00000000", self-user-id: "126137", self-username: "TealIbis540"}
game_id: 935754635
game_result: {roll: 5.2262, win: 0, total_bet: 0, multiplier: 1}
player_seed: "e40014bb8b1fe14ea668211882a826b3bb89fb89c608244682d86bdc9328bea2|0"
result: "0cc264112d7bfb69319484cf7eff2687267e0f793867cf39bd397c4fbdbe0e6f8ac7e426b2ba5663f0169151a57a97525768e35af6c01e294819d0aba7f555a5"
server_hash: "4d7a90e90fe4843e0e1ff9f8dbcc52cfba43bbab4251fd1efc2cac51cbb2c7b1"
success: true

2) act : new_server_seed

data passed

token: 4ug2IKQd0tXPRX
secret: 0
act: new_server_seed


reponse: new server seed hash + previous server seed

{success: true, server_seed: "49118234688c49a6a9b4ce71c47dab6c6ce3347e8450565f1e4d091aeb230e29",…}
server_hash: "4d7a90e90fe4843e0e1ff9f8dbcc52cfba43bbab4251fd1efc2cac51cbb2c7b1"
server_seed: "49118234688c49a6a9b4ce71c47dab6c6ce3347e8450565f1e4d091aeb230e29"

if server seed is changed client seed should change also

Fetch result endpoint

fetch on bet id

Request URL: https://bitvest.io/fetch_result.php
Request Method: POST
Status Code: 200 

from data : game=dice&id=941543545

result: 

result-bettor: "devilla"
result-game-bet: "0.00000000"
result-game-date: "Sep. 2, 9:08:44"
result-game-id: "941,543,545 - BTC"
result-game-name: "Dice"
result-game-profit: "0.00000000"
result-game-won: "0.00000000"
result-section-1: "2.2748"
result-section-2: "&le; 49.9999"
result-section-3: "1.980x"
result-title-1: "Roll"
result-title-2: "Target"
result-title-3: "Payout
Getting recent bets with seeds info

https://bitvest.io/update?games_only=1


Result endpoint

https://bitvest.io/results?query=${item.id}&game=dice&json=1


Stake / PrimeDice
Stake & PrimeDice both use the exact same graphql APIs since they are both developed by the same team.

Request URL (stake) : https://api.stake.com/graphql
Request URL (primedice) : https://api.primedice.com/graphql

Authentication:

Uses JWT token to pass on with the requests which can be obtained on the operator settings.
JWT token example

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiI5MzI0NmMzMS1mY2RjLTRjZTctOWZiYi00NmE1MmM5MDFjNGIiLCJzY29wZXMiOlsiYmV0Il0sImlhdCI6MTU2NzA2MDkwMywiZXhwIjoxNTcyMjQ0OTAzfQ.5SLbHGZpuYU0WljtOGayOgj4DB_26cmd9xi_r2BZdfY

We would also use usernames for getting bets specific to the user.

Below is list of graphql queries



1) Bet List Query (lookup a players bet history)  [Unrestricted]

{
  user(name: "Dan") {
    houseBetList(limit: 50, offset: 0) {
      id
      iid
      bet {
        ... on CasinoBet {
          game
          payout
          amountMultiplier
          payoutMultiplier
          amount
          currency
          createdAt
        }
      }
    }
  }
}


2. Bet Info Query (lookup a players bet history)  [Unrestricted]

bet(iid: "house:6645313440") {
    id
    iid
    bet {
      ... on CasinoBet {
        game
        payout
        payoutMultiplier
        createdAt
        currency
        user {
          name
        }
      }
    }
  }
  
3. Roll Dice (play stakes dice game) [Authenticated]

mutation {
  diceRoll(amount: 1, target: 50, condition: above, currency: doge) {
    id
    payout
    amountMultiplier
    payoutMultiplier
    createdAt
    nonce
  }
}

client seed 
next server seed hash + unhashed 


4. next server seed

`mutation RotateServerSeedMutation {
          rotateServerSeed {
            id
            seedHash
            nonce
            user {
              id
              activeServerSeed {
                id
                seedHash
                nonce
                __typename
              }
              __typename
            }
            __typename
          }
        }`


Request URL: https://api.stake.com/graphql
Request Method: POST


5. Client seed mutation

Request URL: https://api.stake.com/graphql
Request Method: POST

`mutation ChangeClientSeedMutation($seed: String!) {
  changeClientSeed(seed: $seed) {
    id
    seed
    __typename
  }
}`


6. Looking up for a user for user seeds

 `query userSeeds($name: String) {

        user(name: $name) {
          id
          activeServerSeed {
            seedHash
          }
          previousServerSeed {
            seed
            seedHash
          }
          activeClientSeed {
            seed
          }
          previousClientSeed {
            seed
          }
        }
      }`


7.  Looking up for one bet about seed & nonce info

  /**Query is for looking up one bet**/
           
          let variables = {
            iid: houseBet.iid
          }
            
           let query7 = `query betQuery($iid: String!) {
            
            bet(iid: $iid) {
              id
              iid
              bet {
                ... on CasinoBet {
                  id
                  nonce
                  game
                  serverSeed {
                    seed
                    seedHash
                  }
                  clientSeed {
                    seed
                  }
                }
              }
            }
          }`

-------------------------------------------------------------------------------------------------------------------

For stake, The RNG result of every game should be equal to:
HMAC_SHA256(ServerSeed,PlayerSeed:Nonce:Cursor)

Nonce always start with 1
(cursor is used when bytes are exhausted to compute results for some games)


For PrimeDice, The RNG result of every dice game should be equal to:
HMAC_SHA512(ServerSeed,PlayerSeed-Nonce)



References




