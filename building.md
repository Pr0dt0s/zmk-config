# Building steps

docker run -it --rm --security-opt label=disable --workdir /zmk-config -v .:/zmk-config -v ./out:/out zmkfirmware/zmk-build-arm:3.5-branch /bin/bash

export "CMAKE_PREFIX_PATH=/zmk-config/zephyr:$CMAKE_PREFIX_PATH"

west build -d /build/left -p -b "nice_nano_v2" -s /zmk-config/zmk/app -- -DSHIELD="lily58_left" -DZMK_CONFIG="/zmk-config/config"

west build -d /build/right -p -b "nice_nano_v2" -s /zmk-config/zmk/app -- -DSHIELD="lily58_right" -DZMK_CONFIG="/zmk-config/config"

west build -d /build/settings_reset -p -b "nice_nano_v2" -s /zmk-config/zmk/app -- -DSHIELD="settings_reset" -DZMK_CONFIG="/zmk-config/config"

cp /build/left/zephyr/zmk.uf2 /out/lily58_left.uf2 && cp /build/right/zephyr/zmk.uf2 /out/lily58_right.uf2 && cp /build/settings_reset/zephyr/zmk.uf2 /out/settings_reset.uf2




cd ~/zephyrpro52840

west build -p auto -b nrf52840dk_nrf52840 samples/bluetooth/hci_usb