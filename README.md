# Sea of Thieves SDK Guide

## Index

* General Information
  * [SDK Dump](#sdk)
  * [FindPattern Signatures](#findpattern-signatures)
  * [Get UAthenaGameViewportClient and PostRender address](#get-uathenagameviewportclient-and-postrender-address)
  * [Get UWord and GameInstance](#get-uword-and-gameinstance)
  * [ULocalPlayer, APlayerController and AAthenaPlayerCharacter from LocalPlayer](#ulocalplayer-aplayercontroller-and-aathenaplayercharacter-from-localplayer)
  * [Fonts](#fonts)
* [Engine Drawing](sites/EngineDrawing.md)
* [Actors and their usage](sites/Actors.md)
* [Functions and their usage](sites/Functions.md)
* [Features](sites/Features.md)
* [Credits](#credits)

## SDK
[SoT SDK Dump](https://github.com/pubgsdk/SoT-SDK)  

## FindPattern Signatures
**v2.0.1**
```
GObjects:
89 0D ? ? ? ? 48 8B DF 48 C1 E3 04 33 D2
offset = address + 2
(UObject::GObjects)address + 6 + offset
\x89\x0D\x00\x00\x00\x00\x48\x8B\xDF\x48\xC1\xE3\x04\x33\xD2
xx????xxxxxxxxx

GNames
48 8B 1D ? ? ? ? 48 85 ? 75 3A
offset = address + 3
((FName::GNames))(*)address + 7 + offset
\x48\x8B\x1D\x00\x00\x00\x00\x48\x85\x00\x75\x3A
xxx????xx?xx

UWorld:
48 8B 0D ? ? ? ? 48 8B 01 FF 90 ? ? ? ? 48 8B F8 33 D2 48 8D 4E
offset = address + 3
(UWorld)(*)address + 7 + offset
\x48\x8B\x0D\x00\x00\x00\x00\x48\x8B\x01\xFF\x90\x00\x00\x00\x00\x48\x8B\xF8\x33\xD2\x48\x8D\x4E
xxx????xxxxx????xxxxxxxx

48 8B 35 ? ? ? ? 33 DB
\x48\x8B\x35\x00\x00\x00\x00\x33\xDB, xxx????xx
UEngine* Engine;
Engine = *reinterpret_cast<UEngine**>(Address + *reinterpret_cast<DWORD*>(Address + 3) + 7);
```

## Get UAthenaGameViewportClient and PostRender address
```cpp
auto AthenaGameViewportClient = UObject::FindObject<UAthenaGameViewportClient>("AthenaGameViewportClient Transient.AthenaGameEngine_1.AthenaGameViewportClient_1");

const size_t PostRenderIndex = 87;
const auto vmtPostRender  = *reinterpret_cast<uintptr_t***>(AthenaGameViewportClient) + PostRenderIndex;
```
#### PostRender hook
```cpp
typedef void(__thiscall *tPostRender)(UGameViewportClient* uObject, UCanvas* Canvas);
tPostRender OriginalPostRender;

// Function that got called at the end of rendering, perfect to draw our overlay
void HookedPostRender(UGameViewportClient* thisPointer, UCanvas* canvas)
{			
    // our code here (Overlay, ESP etc.)
    OriginalPostRender(thisPointer, canvas);
}
```

## Get UWord and GameInstance
```cpp
auto uWord = AthenaGameViewportClient->World;
auto gameInstance = AthenaGameViewportClient->GameInstance;
```

## ULocalPlayer, APlayerController and AAthenaPlayerCharacter from LocalPlayer
#### ULocalPlayer
```cpp
auto localPlayer = AthenaGameViewportClient->GameInstance->LocalPlayers[0];
```

#### APlayerController
```cpp
// AthenaGameViewportClient->GameInstance->LocalPlayers[0]->PlayerController
auto playerController = localPlayer->PlayerController;
```
#### AAthenaPlayerCharacter
```cpp
// AthenaGameViewportClient->GameInstance->LocalPlayers[0]->PlayerController->Pawn
auto localPlayerActor = (AAthenaPlayerCharacter*)playerController->Pawn;
// or
auto localPlayerActor = (AAthenaPlayerCharacter*)playerController->K2_GetPawn();
```
## Fonts
Available fonts / UFont names
```
Font Engine.Default__Font
Font Roboto.Roboto
Font RobotoDistanceField.RobotoDistanceField
Font RobotoTiny.RobotoTiny
Font RobotoMono.RobotoMono
Font MapFont.MapFont
Font RiddleMapFont.RiddleMapFont
Font Windlass.Windlass
Font PerfCounterFont.PerfCounterFont
```
To get a font:
```cpp
auto font = UObject::FindObject<UFont>("Font Roboto.Roboto");
```


## Credits
Name | Reason
---- | ---------
igromanru | SDK Dump, this guide and most infromation