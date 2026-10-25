name: CleanYourTV 2.7.0 Onn 4K - dock 6 - prueba

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  crear-apk:
    runs-on: ubuntu-24.04
    timeout-minutes: 35
    steps:
      - name: Obtener archivos del repositorio
        uses: actions/checkout@v4

      - name: Comprobar el ZIP personalizado y extraer fuente
        shell: bash
        run: |
          set -euo pipefail
          python3 - <<'PYCODE'
          import hashlib, pathlib, zipfile
          p = pathlib.Path('cleanyourtv-2.7.0-ONN-DOCK6.zip')
          if not p.is_file():
              raise SystemExit('Falta el ZIP en la raiz del repositorio.')
          digest = hashlib.sha256(p.read_bytes()).hexdigest()
          expected = '2751c209ba35ce274d72ddb03db38f0962c1d177621522dcf5221a6e03a9a15d'
          if digest != expected:
              raise SystemExit('El ZIP no coincide con la fuente verificada. SHA256=' + digest)
          with zipfile.ZipFile(p) as archive:
              if archive.testzip() is not None:
                  raise SystemExit('ZIP corrupto.')
              for item in archive.infolist():
                  path = pathlib.PurePosixPath(item.filename)
                  if path.is_absolute() or '..' in path.parts or path.parts[0] != 'cleanyourtv-2.7.0':
                      raise SystemExit('Ruta inesperada dentro del ZIP: ' + item.filename)
              archive.extractall('.')
          print('Fuente 2.7.0 autenticada por SHA256 y extraida.')
          PYCODE

      - name: Preparar Java 17
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'

      - name: Preparar SDK Android
        uses: android-actions/setup-android@v4

      - name: Instalar plataforma Android 34
        shell: bash
        run: |
          set -euo pipefail
          sdkmanager 'platforms;android-34' 'build-tools;34.0.0'

      - name: Compilar 2.7 como aplicacion paralela sin reemplazar 2.5
        working-directory: cleanyourtv-2.7.0/launcher
        shell: bash
        run: |
          set -euo pipefail
          chmod +x gradlew
          ./gradlew --no-daemon --stacktrace --console=plain \
            -PlauncherApplicationId=com.gathin.tvhome.v27 assembleRelease

      - name: Verificar firma, version y paquete
        shell: bash
        run: |
          set -euo pipefail
          mkdir -p salida
          APK=cleanyourtv-2.7.0/launcher/app/build/outputs/apk/release/app-release.apk
          test -s "$APK"
          AAPT="$ANDROID_HOME/build-tools/34.0.0/aapt"
          SIGNER="$ANDROID_HOME/build-tools/34.0.0/apksigner"
          "$SIGNER" verify --verbose --print-certs "$APK" > salida/FIRMA.txt
          "$AAPT" dump badging "$APK" > salida/MANIFIESTO.txt
          grep -q "package: name='com.gathin.tvhome.v27'" salida/MANIFIESTO.txt
          grep -q "sdkVersion:'26'" salida/MANIFIESTO.txt
          grep -q "versionName='2.7'" salida/MANIFIESTO.txt
          cp "$APK" salida/CleanYourTV_2.7_ONN_DOCK6_PRUEBA.apk
          (cd salida && sha256sum CleanYourTV_2.7_ONN_DOCK6_PRUEBA.apk > SHA256.txt)
          printf '%s\n' \
            '2.7.0 - prueba paralela' \
            'Paquete: com.gathin.tvhome.v27' \
            'No reemplaza com.gathin.tvhome (CleanYourTV 2.5).' \
            'No modifica el lanzador HOME de la TV.' \
            'Dock: Netflix, Xuper Hydra, TizenTube, Apple TV, Max, WiwyXuper.' \
            'Firma de prueba: la clave privada no se entrega. Futuras actualizaciones requeriran firma estable.' \
            > salida/LEEME.txt

      - name: Publicar APK para revision (sin instalar en TV)
        uses: actions/upload-artifact@v4
        with:
          name: CleanYourTV_2_7_ONN_DOCK6_PRUEBA
          path: salida/
          retention-days: 14
          if-no-files-found: error
