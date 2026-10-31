# FORM / Shelf — DomCraft experiment

Browser-based furniture configurator with a real open-compartment 3D model.
Dimensions, row and column counts and finishes update the model. Orbit with
the pointer and zoom with the wheel. Export a PNG view or JSON configuration;
shared links restore bounded, validated settings. No furniture orders or
payments are processed.

Run `npm ci` and `npm run generate`. Publish `.output/public/` as static files.
`npm run dev` starts the development server. The renderer uses demand rendering
and disposes geometry, materials, controls and resize observers on unmount.
WebGL-unavailable visitors can still edit and download configuration parameters.

Fonts: Manrope under the SIL Open Font License in public/fonts/.
