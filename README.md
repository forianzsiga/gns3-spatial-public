# Projekt dokumentáció: XR hálózat-szimulációs környezet és GNS3 integráció, 3DGS rekonstrukcióval

> **Beüzemelés:** [SETUP.md](SETUP.md), amely tartalmazza az előfeltételeket, a build folyamatot és a futtatást.

## 1. A projekt célkitűzése
A projekt egy XR (kiterjesztett valóság) alapú frontend fejlesztése a GNS3 hálózati szimulátorhoz. A felhasználó egy 3D/VR virtuális laborban, izommemóriára építve gyakorolhatja routerek, switchek és egyéb hardverek beépítését, kábelezését és konfigurálását. A koncepció bármilyen térbeli interakciót igénylő szimulációra bővíthető, például gyártósori műveletekre, orvosi beavatkozásokra vagy tűzoltási szimulációkra, ahol az ösztönszerű tájékozódás és gyors reagálás kulcsfontosságú.

Másik kutatási irány a valós terek 3D-s rekonstrukciója 3D Gaussian Splatting módszerrel, hiányos referenciaképek alapján. Ehhez a szakdolgozati részben világmodelleket és nézetszintézisre képes diffúziós modelleket alkalmaznék, a jelenlegi önálló laboratóriumi követelményeken túlmutatóan.

## 2. Megvalósított funkciók és jelenlegi állapot

A projekt fejlesztés alatt áll, de a kritikus elemek már működnek. Az alábbiakban a főbb komponenseket és állapotukat részletezem.

### 2.1. GNS3 szerver szinkronizáció és middleware
A backend és frontend közötti kommunikáció működik.
* **Állapotszinkronizáció:** A Godot XR kliens (`gns3_sync_manager.gd`, `gns3_websocket_client.gd`) WebSocket-en keresztül szinkronizál a GNS3 szerverrel: node-ok és kapcsolatok állapota, valamint a szerver eseményei érkeznek a kliensre.
* **API Middleware:** A GNS3 REST API és WebSocket értesítések felett egy köztes réteg a fizikai interakciókat (eszköz elindítása, kábel bedugása) GNS3 parancsokká alakítja.

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">GNS3 szerver szinkronizáció (Videó)</summary>
<hr style="border-color:#30363d;">

<video src="https://github.com/user-attachments/assets/abe21591-37d0-4219-9cb7-2110470fa644" 
autoplay 
loop 
muted 
playsinline 
controls="false" 
width="100%"></video>

<br><i>Ábra 1: gns3-server és gns3-spatial szinkronizáció</i>

</details>

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">Szinkronizáció részletei (Videó)</summary>
<hr style="border-color:#30363d;">

<img src="https://github.com/user-attachments/assets/a6d1f6cb-f5cb-4613-bcf4-c0d319938ef3" 
alt="GNS3 szerver szinkronizáció" 
width="100%">

<br><i>Ábra 2: szinkronizáció részletei (WebSocket debugging a konzolon)</i>

</details>

### 2.2. Fizika, UX és kábelezés (interakció)
A térbeli élmény része a valósághű fizika és az intuitív térbeli felhasználói élmény, vagyis az UX.
* **Kábelfizika:** A projekt egy saját 3D-s kábel-fizikai modellt (`cable_3d.gd`, `physics_orchestrator.gd`) használ, amely élethűen szimulálja a patch-kábelek lógását (lásd az 5. ábrát).
* **Vizuális Visszajelzések:** A virtuális térben megtalálhatók általános, parametrizálható hálózati eszközök (device spawner, drawer), amelyekkel az interakciót árnyalók és további vizualizációk (`fresnel_glow.gdshader`, `port_hover_indicator.gd`) segítik (lásd az 6. ábrát).
* **Intuitív UX és Térbeli Monitorok:** A rendszer fejlett UX megoldásokat alkalmaz. Jó példa erre, hogy a felhasználó egy fiókból húzhatja elő a monitort, majd a számára legkényelmesebb módon dokkolhatja maga elé az adott hálózati eszközt. Jelenleg a hagyományos 2D-s GUI elemek csak monitoros nézeten keresztül érhetők el; az XR UI integráció a hand trackinggel együtt várható (lásd a 2.3 pontot).

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">Kábel fizika demó (Videó)</summary>
<hr style="border-color:#30363d;">

<video src="https://github.com/user-attachments/assets/5a67f482-703c-472e-b06a-6398a5cb0534" 
autoplay 
loop 
muted 
playsinline 
controls="false" 
width="100%"></video>

<br><i>Ábra 5: 3D kábel-fizika szimuláció több switch között, Verlet integrációval és állítható merevséggel</i>

</details>

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">Device spawner fiók demó (Videó)</summary>
<hr style="border-color:#30363d;">

<video src="https://github.com/user-attachments/assets/0b16430a-f15a-453a-8886-e39c249dc782" 
autoplay 
loop 
muted 
playsinline 
controls="false" 
width="100%"></video>

<br><i>Ábra 6: Device spawner fiók, hálózati eszközök drag and drop elhelyezése a virtuális térben</i>

</details>

### 2.3. Optimalizált mélytanulási megoldások és jövőbeli UX víziók (hand tracking)
A térbeli megjelenítéshez a Godot 4 OpenXR implementációját alkalmazza a rendszer, amely monitoros használathoz fallback réteget is biztosít.
* **Kézkövetés (Hand Tracking):** A rendszer a Google MediaPipe Hand Landmarker modellt használja[^1], amely a Pixel 8 Tensor TPU-ján fut. A `gns3-6dof-arcore-companion-app` modul valós idejű kézkövetést végez az Android/ARCore companion alkalmazásban, a csontváz landmarkok 3D koordinátáit a helyi GPU-ról kiolvasva. A cél a jövőbeli, gesztusalapú interakció: csippentés, force pull és térbeli GUI vezérlés.
* **Hibrid Interakció Szegmentációval:** A hálózati terminálok kezelése gyors gépelést igényel. A jövőbeli tervek között gépi látás alapú szegmentációs modellek integrálása szerepel, amelyek a felhasználó fizikai billentyűzetét detektálják és passthrough módban jelenítik meg a virtuális térben.
* **Hálózati VR Streaming és Terheléselosztás:** Az Android/ARCore companion alkalmazás a feldolgozott 6DoF és kézkövetési adatokat Wi-Fi 6 hálózaton streameli egy asztali host gépre, a `freetrack_injector.py` komponensen keresztül. Mivel egy okostelefon nem tud egyszerre teljes OpenXR környezetet, ARCore-t, ML kézkövetést és szegmentációt futtatni, a megoldás feladatmegosztáson alapul: a Pixel 8 TPU-ja a kézkövetést végzi, a hálózaton keresztüli renderelés-kiszervezés pedig alacsony késleltetést és közeli valós idejű VR visszajelzést biztosít. A kliensoldali és szerveroldali számítási terhelés elosztása jelenleg optimalizálás alatt áll; a teszteléshez VRidge-et használunk a telefon és a PC között, a kézkövetés pedig alapvetően a szoftver validálására szolgál, drága headset (pl. Pico) nélkül. A késleltetés a költségkerethez képest meglepően alacsony és nem okoz jelentős hányingert.

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">Companion applikáció demó (Videó)</summary>
<hr style="border-color:#30363d;">

<video src="https://github.com/user-attachments/assets/06e1d1f0-5c46-407c-a197-4a98fbb10ee3" 
autoplay 
loop 
muted 
playsinline 
controls="false" 
width="100%"></video>

<br><i>Ábra 3: A Companion App overlay a Godot XR frontend felett futó sztereoszkópikus VR nézetben</i>

</details>

<details style="border:1px solid #30363d; border-left:4px solid #58a6ff; border-radius:6px; padding:10px 14px; margin:12px 0; background:#161b22;">
<summary style="cursor:pointer; font-weight:bold; font-size:1.05em; color:#58a6ff;">Kéz követés demó (Videó)</summary>
<hr style="border-color:#30363d;">

<video src="https://github.com/user-attachments/assets/2b3e3105-676e-4db9-afbe-910ee6063e3b" 
autoplay 
loop 
muted 
playsinline 
controls="false" 
width="100%"></video>

<br><i>Ábra 4: A hand tracking panel kibontott állapota és ARCore Live Tracker statisztikák (offloading az eszköz TPU-ján), kamera előnézet a bal kéz landmark csontvázával</i>

</details>



### 2.4. Fejlett térbeli rekonstrukció (3DGS, terv)
A fizikai telepítési környezet digitalizálása a projekt legmodernebb iránya. Szerverszobákból, laborokból és irodákból készített sparse referenciaképek alapján neurális rekonstrukciós pipeline hozna létre 3D Gaussian Splat modelleket. Ezzel a hálózat-szimuláció a valós fizikai tér digitalizált másában valósulhatna meg, nem pedig egy steril virtuális térben. A pipeline jelenleg fejlesztési fázisban van; a részleteket a *Jövőbeli tervek* szakasz tartalmazza.

## 3. Rendszerarchitektúra és komponensek
A projekt moduláris felépítésű, négy fő komponensből áll:
1. **XR Frontend (Godot 4):** Térbeli renderelés, kábel-fizika és felhasználói interakciók kezelése.
2. **GNS3-Server és Middleware:** Hálózati rétegek virtualizációja és eszközök logikai összekötése.
3. **ARCore Companion App / OpenVR Driver:** A Pixel 8 szenzoros és ML számításainak begyűjtése, majd streaming az asztali VR környezetbe.
4. **Gaussian Splatting Pipeline:** Terv; a részleteket a *Jövőbeli tervek* szakasz tartalmazza.

<img src="https://github.com/user-attachments/assets/e0fb9925-94e5-4664-a215-b93eb055ec98" 
alt="Rendszerarchitektúra diagram" 
width="100%">

<br><i>Ábra 7: Rendszerarchitektúra; adatfolyam a Pixel 8-tól a VR headset-ig, a jobb oldali nézet a GNS3 integrációt mutatja</i>

## Önálló laboratórium követelményeinek való megfelelés

A projekt technológiai mélysége meghaladja a minimum elvárásokat. A BME-n az Önálló laboratórium tárgy célja, hogy a hallgatók kutatási, tervezési és fejlesztési folyamatokat sajátítsanak el egy komplex probléma önálló megoldásán keresztül. Az Önlab nem kész szoftverterméket vár el, hanem **kutatási eredményeket, mérnöki döntések alátámasztását (PoC) és komplex problémák megoldását.**

1. **Több tudományterületet átívelő:** Jelfeldolgozás/gépi tanulás (Pixel 8 Tensor TPU, kézkövetés), 3D számítógépes grafika (OpenXR, Godot, shader programozás, fizika), elosztott rendszerek (streaming késleltetés-optimalizáció) és hálózati virtualizáció (GNS3 protokollok) egyszerre.
2. **Kutatási érték:** A jelfeldolgozási pipeline szétbontása és egy mobileszköz TPU-ján (Edge AI) végzett inferencia, majd az adatok hálózaton keresztüli streamelése, illetve a késleltetés és tehermentesítés kompromisszumainak vizsgálata önmagában elegendő önálló labor témának.
3. **Korszerű technológiák:** A 3D Gaussian Splatting beemelése, különösen a sparse-image alapú novel view diffúziós modellek, a számítógépes látás jelenlegi legfrissebb trendje.
4. **Tiszta architektúra:** A forráskód refactorált elemei (pl. `HardwareIntegrationManager`) szoftvertervezési és architekturális (SOLID) alapelveket követnek.

## Korlátok

* **Kézkövetés pontossága:** A MediaPipe alapú kézkövetés nem éri el az ARCore natív hand trackingjának pontosságát, de elegendő a gesztusalapú interakció PoC szintű validálásához.
* **Zero-budget VR:** A VRidge streaming a telefon és a PC között nem biztosít olyan minőséget, mint egy dedikált VR headset (pl. Pico 4, Meta Quest 3), de a szoftver teszteléséhez és a koncepció igazolásához megfelelő.
* **Késleltetés:** A Wi-Fi 6 hálózaton mért késleltetés a költségkerethez képest meglepően alacsony és nem okoz jelentős hányingert, így a VRidge + telefon + kézkövetés kombináció vállalható kompromisszum.
* **Kliens-szerver terhelés:** A mobileszköz és az asztali gép közötti számítási terhelés elosztása jelenleg optimalizálás alatt áll; a jelenlegi megoldás elsősorban a szoftver validálására szolgál.

## Jövőbeli tervek

* **3D Gaussian Splatting Pipeline:** Sparse-view rekonstrukció neurális diffúziós modellekkel; a cél a valós szerverszobák digitalizált másának létrehozása.
* **XR UI integráció:** A hagyományos 2D-s GUI elemek (konfigurációs panelek, terminálok) beemelése a virtuális térbe, a monitoros nézet kiváltása.
* **Gesztus alapú interakció:** Csippentés, force pull és egyéb kézmozdulatok a virtuális eszközök kezelésére.
* **Gépi látás alapú szegmentáció:** A felhasználó fizikai billentyűzetének detektálása és passthrough megjelenítése a VR környezetben.
* **Kiterjesztett architektúra:** A jelenlegi GNS3 integráció bővíthető más szimulátorokkal (pl. Cisco Packet Tracer, Eve-NG) vagy más térbeli interakciót igénylő szimulációkkal (gyártósor, orvosi beavatkozás).

## LLM-ek használata a fejlesztés során

A projekt fejlesztésében nagymértékben vettem igénybe nagy nyelvi modelleket (LLM-eket), az alábbi területeken:

* **Scaffolding és MVP:** Az egyes komponensek kiindulási vázát (pl. WebSocket kliens, UDP streaming, TFLite inferencia pipeline, Godot scene hierarchy) LLM-ek generálták. Ezek a kiindulási pontok gyorsan működő prototípusokat adtak, amelyekre a tényleges implementáció épülhetett.
* **Refaktálás és kódminőség:** A `tools/` mappában lévő automatizált compliance scriptek (indentáció, függőségi coupling, LOC analízis) LLM-ek segítségével készültek, és folyamatosan fenntartják a kód alapvető minőségét.
* **Dokumentáció és kommentálás:** A README, a build scriptek és a belső dokumentáció LLM-ek által generált szövegeken alapulnak, amelyeket emberi felülvizsgálat után véglegesítettem.
* **Kézi átnézés:** A scaffolding és az automatizált refaktorálás után a teljes kódbázist manuálisan átnéztem architekturális problémák szempontjából. A kódban található stubok, TODO-k és mock adatok szándékosak, és a jelenlegi fázisnak megfelelően jelölik a még nem implementált, vagy a teszteléshez szükséges részeket.

Ugyanakkor a projekt komplexitása meghaladja az LLM-ek önálló megoldási képességét. A hardver-specifikus integráció (Tensor TPU NNAPI delegáció, ARCore 6DoF streaming, OpenXR action map), a GNS3 API protokollok sajátosságai (binary-only WebSocket, uBridge portkezelés) és a valós idejű késleltetés-optimalizáció olyan területek, ahol az LLM-ek csak mély beavatkozással és folyamatos emberi kontrollal működnek.

---

## Hivatkozások

[^1]: Google MediaPipe Hand Landmarker, elérhetőség: [GitHub](https://github.com/google-ai-edge/mediapipe) · [Hivatalos dokumentáció](https://developers.google.com/mediapipe/solutions/vision/hand_landmarker). A szükséges modelleket (`hand_detector.tflite`, `hand_landmarks_detector.tflite`) a Google MediaPipe hivatalos repozitóriumából gyűjtöttük, majd átportoltuk a Pixel 8 Tensor processzorára NNAPI delegáción keresztül.
  - **NNAPI delegate:** `TpuTrackingEngine.kt`
  - **Pipeline:** `FloatingTrackingService.kt`
  - **Build/deploy:** `build_and_deploy.py`
  - **Modellek:** a `build.gradle` fájl alapján TensorFlow Lite 2.14.0 + MediaPipe Tasks 0.10.20 verziókat használunk
  - **Desktop renderelés:** `hand_visualizer.gd`
