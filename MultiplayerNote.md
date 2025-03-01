# Level Travel

## 2 Ways of travel

### Non seamless travel
- Client disconnects and reconnects to the same server
- Occurs when loading a map or connecting to the server for the first time & when ending a multiplayer game and starting a new one

### Seamless travel
- Smoother experience and avoid reconnection issues
- Enabled on the game mode: `bUseSeamlessTravel = true;`
- Need a transition map because a map must always be loaded at any given time.

## Travel in Multiplayer

### `UWorld::ServerTravel`
- Server Only
- Jumps the server to a new level
- All connected clients will follow
- Server calls `APlayerController::ClientTravel`

### `APlayerController::ClientTravel`
- When called from a client: travel to new server
- When called from a server: makes the player travel to a new map

# Network Role

## Local Role

- Determine how much control the local machine has over this actor
- If Role property is `ROLE_Authority`, then this running instance of **Unreal Engine (UE)** is in charge of this actor (whether it's replicated or not).

## Remote Role
- Determine how much control a remote machine has over this actor

## Matrix of Roles for Client-Server

|**Local Role**|**Remote Role**|**Server or Client**|**Description**|
|---|---|---|---|
|`ROLE_Authority`|`ROLE_AutonomousProxy`|Server|This actor pawn is controlled by one of the connected clients.|
|`ROLE_AutonomousProxy`|`ROLE_Authority`|Client|This actor pawn is controlled by this connected client.|
|`ROLE_SimulatedProxy`|`ROLE_Authority`|Client|This actor pawn is controlled by one of the other connected clients.<br><br>Replicated actors that are not controlled by any client can also have this role combination.|
|`ROLE_Authority`|`ROLE_None`|Client|This is a non-replicated actor.|

# Replication

## Replicated Variables

- `UPROPERTY(Replicated)`
- Override `GetLifetimeReplicatedProps`
- Register Property using macro `DOREPLIFETIME`

## Actor creation and destruction

- When Actor is created and explicitly destroyed, this will automatically be propagated to other clients.
- Some logic can be written in `Destroyed()` function to avoid calling RPCs.
- But notice if the creation and destruction happens in the same tick, `Destroyed` will not be called. (e.g. Spawned a bullet that flies too fast)

## Pitch in multiplayer

Pitch is compressed to an `uint16` type during networking:
```
template<typename T>  
FORCEINLINE uint16 TRotator<T>::CompressAxisToShort( T Angle )  
{  
    // map [0->360) to [0->65536) and mask off any winding  
    return FMath::RoundToInt(Angle * (T)65536.f / (T)360.f) & 0xFFFF;  
}
```
So in client a pitch in range of [-90, 0] will be mapped to [270, 360]

# Remote Procedure Calls

- Functions calls from one machine and executes on another machine
- Macro: `UFUNCTION(Server, Reliable)`

# Net Update Frequency
- Set up update frequency of an actor:
```
NetUpdateFrequency = 66.0f;  
MinNetUpdateFrequency = 33.0f;
```

- Set up server tick frequency in DefaultEngine.ini

```
[/Script/OnlineSubsystemUtils.IpNetDriver]  
NetServerMaxTickRate=60
```



# RPC

## Types of RPC

| **Metadata Specifier** | **Description**                                                                                                                                                                                                                                                         |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Client`               | The RPC is executed on the owning client connection for this actor.                                                                                                                                                                                                     |
| `Server`               | The RPC is executed on the server.<br><br>Must be called from the client that owns this actor.                                                                                                                                                                          |
| `NetMulticast`         | The RPC is executed on the server and all currently connected clients the actor is relevant for.<br><br>`NetMulticast` RPCs are designed to be called from the server, but can be called from clients. A `NetMulticast` RPC called from a client only executes locally. |

## Matrix of RPC Execution

### Server RPC

|**Calling Machine**|**Owning Connection**|**Executing Machine**|
|---|---|---|
|Server|Client|Server|
|Server|Server|Server|
|Server|None|Server|
|Client|Invoking Client|Server|
|Client|Different Client|Dropped|
|Client|Server|Dropped|
|Client|None|Dropped|

### Client RPC

|**Calling Machine**|**Owning Connection**|**Executing Machine**|
|---|---|---|
|Server|Owning Client|Owning Client|
|Server|Server|Server|
|Server|None|Server|
|Client|Invoking Client|Invoking Client|
|Client|Different Client|Invoking Client|
|Client|Server|Invoking Client|
|Client|None|Invoking Client|

### Net Multicast RPC

|**Calling Machine**|**Owning Connection**|**Executing Machine**|
|---|---|---|
|Server|Client|Server and all Clients the invoking actor is relevant for|
|Server|Server|Server and all Clients the invoking actor is relevant for|
|Server|None|Server and all Clients the invoking actor is relevant for|
|Client|Invoking Client|Invoking Client|
|Client|Different Client|Invoking Client|
|Client|Server|Invoking Client|
|Client|None|Invoking Client|

## Reliability

RPCs in Unreal Engine are marked as either reliable or unreliable:

| **Metadata Specifier** | **Description**                                                                                                                           | **Order of Execution** |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| `Reliable`             | This RPC is re-sent until it is acknowledged by the receiver. All subsequent RPC executions are suspended until this RPC is acknowledged. | Guaranteed in order.   |
| `Unreliable`           | This RPC is not executed if the packet is dropped.                                                                                        | No order guarantee.    |





# Game Framework

## Game Mode
-  Server only
-  Set Default Classes
	- Pawn
	- Player Controller
	- HUD
-  Rules of Game
	- Player Eliminated
	- Respawning Players
-  Match State
	- Warmup Time
	- Match Time
## Game State
- Server and all Clients
- State of the Game
- Array of Player States
## Player State
- Server and all Clients
- State of the Player
	- Score
	- Defeats
	- Team
## Player Controller
- Server and Owning Client
- Access to The HUD
## Pawn
- Server and all Clients
## HUD / Widgets
- Client only

![image](https://github.com/user-attachments/assets/7977fe40-5e35-4b5b-bfd3-3a7d24f5ad86)


# Gameplay Ability System

## Replication Mode
![image](https://github.com/user-attachments/assets/09eecd84-5b0a-42c2-8cce-7fd3f389f199)


