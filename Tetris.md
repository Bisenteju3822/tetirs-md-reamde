TetrisV2 WebSocket API Documentation
Document Control
Field	Value
Service Name	TetrisV2 Game Service
API Group	Real-time Multiplayer
Version	v2.1
Status	Active
Author	Yash Narela
Last Updated	2025-07-20
Connection Information
Environment	URL	Protocol
Production	ws://192.168.1.204:5001	WS
Development	ws://localhost:5001	WS
Message Structure
All messages follow this JSON format:

json
{
  "type": "event_type",
  "data": {}, // Event-specific payload
  "error": {  // Only present for error responses
    "code": "ERROR_CODE",
    "message": "Error description" 
  }
}
Event Catalog
1. Room Management Events
join_room (Client → Server)
json
{
  "type": "join_room",
  "data": {
    "uniqueId": "1749703909403",
    "name": "TestPlayer",
    "matchOptionId": "68525b688ef9800405539081",
    "userId": "65d4f8a9e4b0c3a9f4e3b2c1",
    "isPrivate": false,
    "allowedUserIds": [],
    "useBonus": false
  }
}
joined (Server → Client)
json
{
  "type": "joined",
  "data": {
    "roomId": "hCFlRZ_io",
    "sessionId": "bPIer1reo"
  }
}
join_room_error (Server → Client)
json
{
  "type": "join_room_error",
  "error": {
    "code": "ROOM_FULL",
    "message": "The room is full"
  }
}
2. Game Events
join_game (Client → Server)
json
{
  "type": "join_game",
  "data": {
    "name": "TestPlayer1",
    "uniqueId": "1749703909403"
  }
}
state (Server → Client)
json
{
  "type": "state",
  "data": {
    "gameStatus": "waiting",
    "winner": "",
    "playerCount": 2,
    "minPlayer": 2,
    "countdown": 0,
    "matchOptionId": "68525b688ef9800405539081",
    "betAmount": 20,
    "winAmount": 40,
    "everyoneJoined": false,
    "players": {}
  }
}
state-update (Server → Client)
json
{
  "type": "state-update",
  "data": {
    "players": {
      "abc456": {
        "id": "abc456",
        "name": "Player1",
        "uniqueId": "player123",
        "score": 0,
        "board": [0,0,0,...], // 100 zeros (10x10 grid)
        "currentPieces": [
          {"rows": [1,1,1,1], "width": 4, "height": 1, "pieceId": 1},
          {"rows": [1,1,1,1], "width": 2, "height": 2, "pieceId": 2},
          {"rows": [1,0,1,1], "width": 2, "height": 2, "pieceId": 3}
        ],
        "gameOver": false
      }
    }
  }
}
3. Game Flow Events
both-player-join (Server → Client)
json
{
  "type": "both-player-join",
  "data": {
    "gameStatus": "starting",
    "countdown": 3
  }
}
countdown-update (Server → Client)
json
{"type": "countdown-update", "data": {"countdown": 3}}
{"type": "countdown-update", "data": {"countdown": 2}}
{"type": "countdown-update", "data": {"countdown": 1}}
game-started (Server → Client)
json
{
  "type": "game-started",
  "data": {
    "gameStatus": "in-progress",
    "everyoneJoined": true
  }
}
4. Game Actions
place_piece (Client → Server)
json
{
  "type": "place_piece",
  "data": {
    "index": 1,
    "x": 3,
    "y": 4
  }
}
board-clear (Server → Client)
json
{
  "type": "board-clear",
  "data": {
    "playerId": "abc456"
  }
}
game-over (Server → Client)
json
{
  "type": "game-over",
  "data": {
    "playerId": "abc456",
    "reason": "inactivity"
  }
}
5. Game Conclusion
game_ended (Server → Client)
json
{
  "type": "game_ended",
  "data": {
    "winner": "def789",
    "scores": [
      {"playerId": "abc456", "name": "Player1", "score": 450},
      {"playerId": "def789", "name": "Player2", "score": 780}
    ]
  }
}
game_ended (Draw)
json
{
  "type": "game_ended",
  "data": {
    "winner": "",
    "isDraw": true,
    "scores": [
      {"playerId": "qhlsMsev7", "name": "TestPlayer", "score": 20},
      {"playerId": "9dHTSMOf1", "name": "TestPlayer", "score": 20}
    ]
  }
}
6. System Events
rematch_request (Client → Server)
json
{"type": "rematch_request"}
exit (Client → Server)
json
{"type": "exit"}
left (Server → Client)
json
{"type": "left", "data": {"code": 1000}}
Error Handling
invalid_move (Server → Client)
json
{
  "type": "invalid_move",
  "error": {
    "code": "INVALID_PLACEMENT",
    "message": "Cannot place piece here"
  }
}
error (Server → Client)
json
{
  "type": "error",
  "error": {
    "code": "ROOM_FULL",
    "message": "Room is full"
  }
}
Field Requirements
Field	Required	Description
uniqueId	Yes	Player identifier for reconnection
name	Yes	Player display name
matchOptionId	Yes	Match configuration ID
userId	If private	Required for private matches
useBonus	No	Flag for bonus usage
isPrivate	No	Private room flag
allowedUserIds	No	List of allowed players
Connection Flow
Client connects to WebSocket

Client sends join_room

Server responds with joined

Client sends join_game

Server sends state updates

Game progresses with place_piece events

Game concludes with game_ended

Reconnection
Players can reconnect using their uniqueId:

json
{
  "type": "join_room",
  "data": {
    "uniqueId": "1749703909403"
  }
}
Error Codes
Code	Description
ROOM_FULL	Room at maximum capacity
INVALID_PLACEMENT	Illegal piece placement
GAME_IN_PROGRESS	Match already started
INACTIVITY_TIMEOUT	Player disconnected too long
MATCH_NOT_FOUND	Invalid match configuration
