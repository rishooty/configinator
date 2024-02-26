# configinator

Less tedious config/cfg/ini/yaml configuration than just plaintext editor alone.

Only tested on retroarch's own .cfg where everything is a string at the moment.

Goal is to have this publicly hosted, as a PWA, and a self hosted version with its own node-express API to edit files directly on emulation handhelds or mini pcs.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## TODO:
[] Have it parse special comments for more validation and input types; i.e. # C[Step: 10, Max: Height]R or C[Type: SDL Input]
[] Have parse nearby comments (excepting the above) as tooltips.
[] Render each group as cascading lists.
[] Add support for mupen64plus.cfg - files with a mix of strings and non strings
[] Add support for zdoom.ini - .ini files where strings are not enclosed in quotes, support for tuples as three number input fields.
[] Add PWA support
[] Host on AWS or Firebase, probably the latter.
[] Add node express api