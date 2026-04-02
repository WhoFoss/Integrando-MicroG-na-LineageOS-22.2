1. Adicionar o repositório do MicroG
```
nano .repo/local_manifests/microg.xml
```

 2. **Cole isso dentro do arquivo:**
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <remote name="lineageos4microg"
            fetch="https://github.com/lineageos4microg/" />

    <project path="vendor/partner_gms"
             name="android_vendor_partner_gms"
             remote="lineageos4microg"
             revision="master" />
</manifest>
```

3. **Sincronizar** 
```
cd ~/los
repo sync vendor/partner_gms
```

---
### HabHabilitar Signature Spoofing em builds user

> [!NOTE]
> Por que esse passo é necessário?
> O MicroG precisa se passar pelo Google Play Services para que os apps funcionem normalmente. Ele faz isso através do Signature Spoofing — um mecanismo que permite ao MicroG fingir ter a mesma assinatura criptográfica do pacote oficial do Google (com.google.android.gms).
> 
> A LineageOS já inclui suporte ao Signature Spoofing no source (ComputerEngine.java), mas por decisão de segurança, ele está bloqueado por uma verificação que só permite seu funcionamento em builds userdebug, não em builds user (produção).
> Para distribuir a ROM com MicroG totalmente funcional em builds user, é necessário remover essa restrição manualmente.

### Patch manual no framework para poder compilar a build como user

1. Abra o arquivo na linha exata:
```
nano +1496 frameworks/base/services/core/java/com/android/server/pm/ComputerEngine.java
```
> [! ATENÇÃO]
> O número da linha pode variar se o source for atualizado. Se necessário, busque pelo conteúdo if (!isDebuggable()) dentro do método isMicrogSigned.

 2. E deleta as linhas 1496, 1497 e 1498.
Depois confirma que ficou assim:
```
public static boolean isMicrogSigned(AndroidPackage p) {
        // Allowlist the following apps:
        // * com.android.vending - microG Companion
        // * com.google.android.gms - microG Services
        if (!p.getPackageName().equals("com.android.vending") &&
```

### As linhas que devem ser deletadas são:
```
if (!isDebuggable()) {
            return false;
        }
```

---

### O resultado final do método deve ficar assim:
```
public static boolean isMicrogSigned(AndroidPackage p) {
        // Allowlist the following apps:
        // * com.android.vending - microG Companion
        // * com.google.android.gms - microG Services
        if (!p.getPackageName().equals("com.android.vending") &&
                !p.getPackageName().equals("com.google.android.gms")) {
            return false;
        }
        return Signature.areExactMatch(
                p.getSigningDetails(), new Signature[]{MICROG_REAL_SIGNATURE});
    }
```

---
### Agora compila:
```
source build/envsetup.sh
lunch lineage_sapphire-bp1a-user
export WITH_GMS=true
mka bacon
```
