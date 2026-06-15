name: Build Android APK
on: [push, workflow_dispatch]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract ZIP
        run: |
          unzip -q minds_meetup_android_studio_ready.zip -d ./extracted_project
          # सही एंड्रॉयड फोल्डर ढूंढने की तरकीब
          TARGET_DIR=$(find . -type d -name "android" | head -n 1)
          echo "FOUND_ANDROID_DIR=$(dirname $TARGET_DIR)" >> $GITHUB_ENV

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install Dependencies
        run: |
          cd ${{ env.FOUND_ANDROID_DIR }}
          if [ -f package.json ]; then
            npm install --legacy-peer-deps
          fi

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Build APK
        run: |
          cd ${{ env.FOUND_ANDROID_DIR }}/android
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: Minds-Meetup-APK
          path: "**/build/outputs/apk/debug/*.apk"
