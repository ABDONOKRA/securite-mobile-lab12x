# securite-mobile-lab12x

## Preuve d’installation


<img width="1168" height="458" alt="image" src="https://github.com/user-attachments/assets/22016891-0cab-44de-8611-8352b2c67290" />  

## verifier l'emulateure  et vérifiez l'installation 

<img width="613" height="188" alt="image" src="https://github.com/user-attachments/assets/d895151c-03ed-4605-b6d8-11c4a803b65b" />

<img width="722" height="182" alt="image" src="https://github.com/user-attachments/assets/e46e5b94-0ded-47a7-be4e-aae797c00a1c" />  

<img width="1073" height="140" alt="image" src="https://github.com/user-attachments/assets/923dfb39-e82a-4b50-b7d9-57b12acad931" />

<img width="893" height="617" alt="image" src="https://github.com/user-attachments/assets/6fb508d3-6b13-4bc3-9571-293e3bd22a11" />  

## Installer Medusa (outil d’instrumentation)  

<img width="893" height="617" alt="image" src="https://github.com/user-attachments/assets/167960e1-e06a-4a4f-932d-8197c7199c09" />
<img width="1857" height="710" alt="image" src="https://github.com/user-attachments/assets/b7f0ee35-909d-45ac-b81b-50d9b8f61775" />

# Lapk cible est uncrackable 3
<img width="441" height="836" alt="image" src="https://github.com/user-attachments/assets/6a897ca8-4bde-47b6-973d-fc3c49e64ed5" />

# ✅ Étape 3 — VALIDÉE
L'application Uncrackable Level 3 détecte le root et affiche :  


[]"Rooting or tampering detected. This is unacceptable. The app is now going to exit."  
# Étape 4 — Lancer le bypass avec Medusa  

<img width="1374" height="606" alt="image" src="https://github.com/user-attachments/assets/346ed101-e81d-43ac-85f2-ffac1c2b4cb9" />

<img width="789" height="776" alt="image" src="https://github.com/user-attachments/assets/6db39791-986b-4e00-86c7-18703e1e6678" />

## Medusa est connecté à l'émulateur !

  
Je remarque quelque chose d'important :  

[ro.build.tags]: [test-keys]  

C'est exactement ce que l'app détecte pour savoir si l'appareil est rooté ! ✅  

Notre cible est le numéro [7] owasp.mstg.uncrackable3.

##  Maintenant cherchons le module root bypass

<img width="746" height="169" alt="image" src="https://github.com/user-attachments/assets/7b2aecc4-8ccf-46e9-b598-769d03bd00c0" />

# Medusa a trouvé 4 modules de root bypass !

---

###  Modules disponibles :

| # | Module | Description |
|---|---|---|
| 1 | `root_detection/universal_root_detection_bypass` | ✅ **Le plus complet** |
| 2 | `root_detection/rootbeer_detection_bypass` | Pour RootBeer spécifiquement |
| 3 | `root_detection/rootbeer_detection_bypass_no_obfuscation` | RootBeer sans obfuscation |
| 4 | `root_detection/jailMonkey_react_native` | Pour apps React Native |

---

On va utiliser le module **universel** qui est le plus adapté à notre cible.  :

```bash
use root_detection/universal_root_detection_bypass
```



 # Module chargé avec succès !
Le module universal_root_detection_bypass est maintenant dans la liste Current Mods.

🚀 Maintenant lançons le bypass !
```bash run ```  
# Une erreure 
<img width="1920" height="697" alt="image" src="https://github.com/user-attachments/assets/86908cd8-4ac9-4bbd-b8fb-9dd52200b18c" />

Frida cherche un gadget arm64 mais notre émulateur est x86_64 ! C'est un conflit d'architecture.
# Solution — Forcer le bon device USB
relancez Medusa en spécifiant explicitement le device USB 


```
python3.12 medusa.py -p owasp.mstg.uncrackable3 -d emulator-5554
```
## Relencement de Medusa  

<img width="1920" height="605" alt="image" src="https://github.com/user-attachments/assets/0e0ab57b-cd99-4264-9b45-90b706be7d3a" />

# Le problème persiste avec le gadget arm64. C'est un bug de Medusa qui cherche le mauvais gadget pour x86_64.    

<img width="1912" height="916" alt="image" src="https://github.com/user-attachments/assets/c2c2fd25-3b5d-4753-a808-589df45837e2" />

  
# 🔧 On passe au Plan B — Frida pur !

<img width="1912" height="916" alt="image" src="https://github.com/user-attachments/assets/8f5c6772-a794-4b31-9e91-648622d0a10c" />

```

// Neutralise Build.TAGS, File.exists, Runtime.exec, RootBeer
function safeContains(str, needle) {
  try { return (str || "").toLowerCase().indexOf((needle||"").toLowerCase()) !== -1; } catch (_) { return false; }
}
const suspiciousPaths = [
  "/system/bin/su", "/system/xbin/su", "/sbin/su", "/system/su",
  "/system/app/Superuser.apk", "/system/app/SuperSU.apk",
  "/system/bin/.ext/.su", "/system/usr/we-need-root/",
  "/system/xbin/daemonsu", "/system/etc/init.d/99SuperSUDaemon",
  "/system/bin/busybox", "/system/xbin/busybox"
];
Java.perform(function () {
  try { const Build = Java.use('android.os.Build'); Object.defineProperty(Build, 'TAGS', { get: function() { return 'release-keys'; } }); console.log('[+] Build.TAGS -> release-keys'); } catch (e) {}
  try { const File = Java.use('java.io.File'); File.exists.implementation = function () { const p = this.getAbsolutePath(); if (suspiciousPaths.indexOf(p) !== -1) { console.log('[+] File.exists bypass for', p); return false; } return this.exists.call(this); }; } catch (e) {}
  try {
    const Runtime = Java.use('java.lang.Runtime'); const JString = Java.use('java.lang.String'); const StringArray = Java.use('[Ljava.lang.String;');
    function blockIfSus(x){ const s = Array.isArray(x)? x.join(' ') : (''+x); const t=s.toLowerCase().trim(); if(t.startsWith('su')||t.includes(' which su')||t.includes(' busybox')||t.includes(' su ')) return ['sh','-c','echo']; return null; }
    Runtime.exec.overload('java.lang.String').implementation = function(cmd){ const r=blockIfSus(cmd); return r? this.exec(JString.$new(r.join(' '))) : this.exec(cmd); };
    console.log('[+] Runtime.exec hooks installed');
  } catch(e) {}
  console.log('[+] Java bypass installed');
});

```




















