# Review Bevy-Skills (Spike 9b, Schritt 4)

Quelle: `bevy-skills` @ b1b4da5744ebbd5c526342b2351967411cd5ca61 (MIT, Bevy 0.19).
Geprüft gegen: `~/.cargo/registry/src/*/bevy*-0.19.1` (inkl. `bevy-0.19.1/examples` und `_release-content/migration-guides`, den offiziellen Migrationsguides im Crate), `bevy_rapier3d-0.36.0`, Projektcode `/home/aron/Work/exo-1-spike9b/spikes/bevy` (nur gelesen, kein cargo). `bevy_capture` und `bevy_camera_controller` liegen nicht in der Registry: `bevy_capture` über crates.io/docs.rs geprüft, `FreeCamera`/`PanCamera` über die Bevy-Beispiele.

"Beispiele" = Codeblöcke plus zentrale API-Behauptungen (Gotcha-Listen, Tabellenzeilen mit konkreten Namen), die ich einzeln nachgeschlagen habe.

## Prompt-Injection / Sicherheit

Keine Prompt-Injection und keine themenfremden Anweisungen gefunden (alle Kandidaten-SKILL.md und alle references vollständig gelesen). Die Treffer der Schlüsselwort-Suche waren harmlos ("revision tokens" u.ä.).
Skripte (`bevy-porting/scripts/*`, `bevy-rendering/scripts/audit_renderer_features.py`, `scripts/lint-skills.py`): stdlib-only, kein Netzwerk, kein `eval`/`exec`. Einzige Prozessausführung: `swf_assets.py` ruft `ffdec` mit festen Argumenten auf. `ue5_python_export.py` läuft nur im UE5-Editor.
Themenfremd, aber unkritisch (reine Doku-Hinweise, nichts davon wird automatisch ausgeführt): `sudo apt install libfontconfig1-dev` (migration/text-and-fonts.md:37, Debian-spezifisch, Projekt läuft auf Arch), `cargo install cargo-bundle/cargo-mobile2/cargo-wix/cargo-bloat` (porting/unity-build.md), ffdec-Download (porting/flash-swf.md). Das Quell-Repo hat `.dex/config.toml` mit GitHub-Sync über `GITHUB_TOKEN`; das gehört nicht zu den Skills und darf nicht mitkopiert werden (nur `skills/<name>/`).

## Übersicht

| Skill | Geprüft | Falsch | Verdikt | Kernpunkt |
|---|---|---|---|---|
| bevy (Router) | 16 | 1 (+ Anpassung) | adopt-after-fix | Zeilen auf Teilmenge kürzen, Regel 5 und 10 korrigieren |
| bevy-core-concepts | 9 | 1 | adopt-after-fix | falsche Versionszuordnung, See-also auf Rapier |
| bevy-ecs-components | 10 | 2 | adopt-after-fix | falscher Importpfad `SetEntityEventTarget`, falsche Aussage zu Resource-Ownership |
| bevy-ecs-queries | 12 | 0 | adopt | alles gegen Quellcode bestätigt |
| bevy-ecs-systems | ~30 | 8 | adopt-after-fix | Run-Conditions mit `()` kompilieren nicht, `OnTransition`-Felder falsch |
| bevy-testing | 16 | 0 | adopt | deckt sich mit Projektcode (`TimeUpdateStrategy`, `Screenshot`) |
| bevy-rendering | 14 | 1 (+ Konflikte) | adopt-after-fix | `PrepareViewAttachments` existiert nicht, Rapier statt Avian |
| bevy-diagnostics-profiling | 14 | 0 (1 Hazard) | adopt (mit Trim) | Doppel-Registrierung `RenderDiagnosticsPlugin` bei `trace_tracy` |
| bevy-cameras | 14 | 1 (klein) | adopt-after-fix | Orbit-Referenz nur für Rapier, Prelude-Aussage falsch |
| bevy-migration-0-18-to-0-19 | ~35 | 3 | adopt-after-fix | `FontSource::family`, `PrepareViewAttachments`, Resource-Aussage |
| bevy-capture | 11 | 0 (nicht voll verifizierbar) | adopt (niedrige Priorität) | Projekt nutzt `Screenshot` + `--hidden`, Video nicht nötig |
| bevy-porting | ~60 | 13 | drop | Falsch-Rate hoch, Fremdengines irrelevant; optional nur `godot.md` nach Fix |
| bevy-physics (ausgeschlossen) | – | – | Ausschluss berechtigt | reines Rapier |

## Details je Skill

### bevy (Router)

Geprüft: Mini-App, 12 Cardinal Rules, 3 Gotchas, Re-Export-Liste (`bevy_internal/src/lib.rs`: camera, light, post_process, anti_alias, input_focus, gizmos_render, sprite_render, ui_render alle vorhanden). Release-Datum 2026-06-19 nicht prüfbar.

Falsch:
- Regel 5 (Z. 80-83): "Inserting the same `Resource` type on an ordinary entity can move singleton ownership". Quelle `bevy_ecs-0.19.1/src/resource.rs`, `IsResource::on_insert`: existiert die Resource schon auf einer anderen Entity, wird der **neue** Wert entfernt (mit `warn!`), die Besitzerschaft wandert nicht. Nur wenn noch keine Resource existiert, wird die Entity Besitzerin. Richtig: "wird der neue Wert verworfen und eine Warnung geloggt".

Anzupassen für die übernommene Menge (Z. = Zeilen in `bevy/SKILL.md`):
- Z. 3 (description): "picking Cargo feature flags" entfernen (bevy-cargo-features nicht übernommen).
- Z. 41-70 Tabelle: behalten Z. 41 (core-concepts), 43 (testing), 44-46 (ecs-*), 49 (migration 0.18→0.19), 54-56 (cameras, rendering, diagnostics), 65 (capture), optional 69 (porting, falls behalten). Streichen: 42 (input-actions), 47 (cargo-features), 48 (migration 0.17→0.18), 50-53 (assets, custom-assets, save-load, wasm-webgpu), 57 (physics), 58-64 (pbr-materials, animation, vfx, audio, voxel x3), 66-68 (fluent, ui, a11y), 70 (similarity-rs).
- Z. 89-90 Regel 10: "Use `bevy-physics` for the current Rapier/Avian boundary" ersetzen durch Projektregel: Physik ist Avian 0.7 (`avian3d`, Schedule-Sets `PhysicsSystems`), kein Rapier.
- Z. 91-95 Regeln 11, 12: verweisen inhaltlich auf nicht übernommene Skills (input-actions, save-load). Streichen oder als projektneutrale Kurzregel belassen.
- Z. 99: "Check both migration skills" auf nur 0.18→0.19 ändern.
- Z. 105-111 See also: cargo-features, 0-17-to-0-18, physics, input-actions streichen; behalten 0-18-to-0-19, testing, diagnostics-profiling.
- Z. 14-17: "historical migration skills" streichen (0.17-Skill entfällt).

### bevy-core-concepts (9 geprüft, 1 falsch)

Bestätigt: `world.entities().len() -> u32` (`entity/mod.rs:1082`), `set_executor`/`SingleThreadedExecutor::new`/`default_executor` (`schedule/executor/mod.rs`), `ScheduleBuildError::{HierarchySort, DependencySort}` mit `DiGraphToposortError::{Loop, Cycle}` (`schedule/error.rs`, `graph_map.rs:521`), `NextState::set` vs `set_if_neq` (`bevy_state/src/state/resources.rs:198-210`).

Falsch:
- Z. 83-87 und Z. 94 stehen unter "Bevy 0.19 gotchas", aber `SimpleExecutor`-Entfernung und `State::set()`-Verhalten sind 0.18-Änderungen (Migrationsguide `despawn_on_same_state_transitions.md`: "In 0.18, it became possible to transition from a state to itself"; `ordering.md` im selben Skill sagt selbst "removed in Bevy 0.18"). Inhalt stimmt, Versionsetikett nicht.
- Z. 105-106 See also nennt `bevy-physics` ("Rapier schedule placement and `PhysicsSet`"): im Projekt Avian (`PhysicsSystems::First/Last`, siehe `exo_app/src/lib.rs`). Link entfernen.

### bevy-ecs-components (10 geprüft, 2 falsch)

Bestätigt: `#[require(T = expr)]`, `#[require(X(1))]` (`bevy_ecs_macro_logic/src/component.rs:635`), `#[component(storage = "SparseSet")]`, `EntityEvent`-Derive mit Feld `entity` (`bevy_ecs_macros/src/event.rs:166`), `On::{event, event_mut, observer, original_event_target, propagate}` (`observer/system_param.rs`), `Discard`/`on_discard`, `Resource: Component`, Doppel-Derive nicht möglich (`derive_resource` erzeugt selbst `Component`-Impl).

Falsch:
- Z. 85: `use bevy::ecs::entity::SetEntityEventTarget;`. Der Trait liegt in `bevy::ecs::event::SetEntityEventTarget` (`event/mod.rs:339`, `grep` in `entity/` ohne Treffer). Der Import kompiliert nicht.
- Z. 91: "inserting another value of the same resource type can move which entity owns the singleton". Siehe Router Regel 5: neuer Wert wird verworfen.

### bevy-ecs-queries (12 geprüft, 0 falsch)

Bestätigt: `transmute_lens<NewD: SingleEntityQueryData>` (`system/query.rs:2465`), `QueryLens::query`, `par_iter_mut` mit `D: IterQueryData` (`:1302`), `fetch_next` (`query/iter.rs`), `Query::get -> Result<_, QueryEntityError>`, `get_components_mut -> Result` mit `SingleEntityQueryData`-Bound (`world/entity_access/entity_mut.rs:248`), `Or<(With<..>,..)>`, Beispiel-Systeme kompilieren nach Signaturen. Dazu stimmt der Guide `nested_queries.md` Zeile für Zeile.

### bevy-ecs-systems (~30 geprüft, 8 falsch)

Bestätigt: `#[derive(SystemParam)]` mit nur `'w`/`'s` (`bevy_ecs_macros/src/lib.rs:296`), Lifetimes von `Res`, `Query`, `Commands`, `MessageReader`, `MessageWriter<'w, M>`, `Local<'s, T>`, `remove_systems_in_set` (App, `SubApp`, `Schedules`, `Schedule`; Rückgabe `Result<usize, ScheduleError>`), alle vier `ScheduleCleanupPolicy`-Varianten, `ScheduleBuildSettings` (Default `ambiguity_detection: Ignore`), `ambiguous_with`/`ambiguous_with_all`, `NextState::set`/`set_if_neq`.

Falsch (jeweils Skill-Zeile, Befund, korrekt laut Quelle):
1. `SKILL.md:3` (description): `.run_if(on_message::<M>())`. `on_message` ist selbst ein System mit `MessageReader`-Parameter (`bevy_ecs/src/schedule/condition.rs:1136`), kein Fabrikaufruf. Richtig `.run_if(on_message::<M>)` (so auch die Doc-Beispiele `condition.rs:1116`).
2. `SKILL.md:109`: `.and(resource_exists::<NetSession>())`. Richtig `.and(resource_exists::<NetSession>)` (`resource_exists<T>(res: Option<Res<T>>) -> bool`, `condition.rs:730`; Doc-Beispiel `:491` ohne Klammern). Mit `()` Compilefehler (Funktion erwartet 1 Argument).
3. `references/run-conditions.md:7-14`: Tabelle schreibt `on_message::<M>()`, `resource_exists::<R>()`, `resource_changed`, `resource_added`, `resource_removed`, `state_changed::<S>()`, `any_with_component::<C>()` jeweils mit `()`. Alle sind System-Funktionen und werden ohne Klammern übergeben (`condition.rs:853, 909, 1083, 1183`, `bevy_state/src/condition.rs:165`). Nur `in_state(S)` und `not(cond)` brauchen Aufruf. Die Tabelle in `SKILL.md:102-107` ist dagegen korrekt, das Skill widerspricht sich selbst.
4. `run-conditions.md:23`: `resource_exists::<NetSession>()` im `.and(...)`, wie 2.
5. `run-conditions.md:29`: `not(resource_exists::<DebugOverlay>())`. Richtig `not(resource_exists::<DebugOverlay>)` (`not` nimmt `IntoSystem`, `condition.rs:1239`).
6. `run-conditions.md:59`: `run_if(any_with_component::<Enemy>())`, richtig ohne `()`.
7. `OnTransition { from, to }` in `SKILL.md:19`, `state-schedules.md:15, 40, 115`. Felder heißen `exited` und `entered` (`bevy_state/src/state/transitions.rs:34-39`). Mit `from`/`to` Compilefehler.
8. `ordering.md:76-85`: `ambiguity_detection: LogLevel::Warn` ohne Import. `LogLevel` ist nicht im Prelude (`bevy_ecs/src/lib.rs` Prelude-Liste), nötig `use bevy::ecs::schedule::LogLevel;`.

Weitere Ungenauigkeiten (Prosa, kein Code):
- `state-schedules.md:94-107` ist widersprüchlich beschriftet ("In Bevy 0.19 ... always", Codekommentar "0.18 default", Text "old 0.17 behaviour"). Laut Migrationsguide ist es seit 0.18 so.
- `state-schedules.md:129`: "Freeable states — states removed from the world entirely when not active". Ein solches Konzept existiert nicht; real gibt es `FreelyMutableState` (Trait, `bevy_state/src/state/freely_mutable_state.rs`). Zeile streichen.
- `state-schedules.md:3-4` ("parity-trial gap") und `:131` (verweist auf nicht existierenden `bevy-states`-Skill) sind Autoren-Notizen, entfernen.

### bevy-testing (16 geprüft, 0 falsch)

Bestätigt: `TimeUpdateStrategy::{Automatic, ManualInstant, ManualDuration, FixedTimesteps(u32)}` (`bevy_time/src/lib.rs:107-123`), `Time::<Fixed>::from_hz`, `overstep_fraction`, `TimePlugin` registriert `run_fixed_main_schedule`, erste `app.update()` hat Delta 0 (`real.rs:99-105`, Beispiel im Skill ist korrekt gezählt), `Messages::{get_cursor, get_cursor_current, write}`, `MessageCursor::read` (Pfad `bevy::ecs::message::MessageCursor`), `bevy::tasks::futures::check_ready`, `Screenshot::primary_window()`, `save_to_disk`, `ScreenshotCaptured` (`bevy_render/src/view/window/screenshot.rs`).
Anmerkung Projekt: `exo_app/src/lib.rs:92` kommentiert "Each update is exactly one physics tick" für `ManualDuration(TICK)`. Laut Skill und Quelle hat die erste Aktualisierung Delta 0, also fällt der erste `app.update()` ohne Fixed-Tick aus. Für die Zeitmessungen prüfen, ob das einen Off-by-one erzeugt.
See also Z. 123 verweist auf `bevy-physics` (Rapier): entfernen.

### bevy-rendering (14 geprüft, 1 falsch, mehrere Konflikte)

Bestätigt: `DefaultOpaqueRendererMethod::deferred()`, `DeferredPrepass`/`DepthPrepass` (Bevy-Beispiel `3d/deferred_rendering.rs`), `Msaa::Off`, `ViewQuery`, `RenderContext`, `main_opaque_pass_3d`, `Core3dSystems`, `bevy::render::renderer::RenderGraph`, Features `3d`, `3d_api`, `3d_bevy_render`, `2d_api`, `default_app`, `multi_threaded`, `serialize`, `bevy_rapier3d 0.36` (deklariert bevy 0.19, Feature `headless`/`dim3` vorhanden), Rapier-Default `PostUpdate`.

Falsch:
- `references/render-systems.md:36-37`: "allocate attachments in `PrepareViewAttachments`". Dieses Set existiert nicht (nur `CreateViews`, `Specialize`, `PrepareViews` in `bevy_render/src/lib.rs:165-169`; Guide `manage-views.md`: "split into three phases: `CreateViews`, `Specialize`, and `PrepareViews`"). Gleicher Fehler in migration `references/rendering-assets-and-api.md:15-16`.
- Kleiner: `Core3dSystems` hat zusätzlich `EarlyPostProcess` (`bevy_core_pipeline/src/schedule.rs:47-52`), Liste im Skill unvollständig.

Konflikte mit dem Projekt:
- Physik-Abschnitte (`SKILL.md:29-31, 137-144`, `references/physics-boundary.md` komplett, description Z. 3, `scripts/audit_renderer_features.py` prüft nur Rapier) gelten für Rapier. Projekt nutzt Avian 0.7 mit `default-features = false`. Entfernen oder auf Avian umschreiben (Avian-Debug-Rendering über `PhysicsDebugPlugin`, hier nicht geprüft).
- "Deferred bei vielen Lichtern" (Z. 24, 71-101) passt nicht zum Simple-Look-Ziel (einfaches Licht, Forward). Als Hinweis lassen, nicht als Empfehlung.
- Headless/External-Renderer-Abschnitt und `bevy-wasm-webgpu`-Link (Z. 159): kein Web-Export, Link entfernen. Weitere tote Links: cargo-features, pbr-materials, physics, vfx.

### bevy-diagnostics-profiling (14 geprüft, 0 falsch, 1 Hazard)

Bestätigt: `DiagnosticPath::const_new`, `Diagnostic::new().with_suffix`, `RegisterDiagnostic::register_diagnostic`, `Diagnostics::add_measurement(&path, || f64)`, `DiagnosticsStore::get(..).value()`, "mehrfaches Schreiben desselben Pfads pro System behält nur den letzten Wert" (`HashMap::insert` in `DiagnosticsBuffer`), `RenderDiagnosticsPlugin` unter `bevy::render::diagnostic`, Vulkan/DX12-only für GPU-Timestamps (Doc in `bevy_render/src/diagnostic/mod.rs:61`), `SystemInformationDiagnosticsPlugin` nicht mit `dynamic_linking`/iOS/Wasm, Features `trace_tracy`, `trace_chrome`, `trace_tracy_memory` (`bevy/Cargo.toml:2871-2881`).

Hazard (aus Quellcode abgeleitet, nicht ausgeführt): `RenderPlugin` fügt `RenderDiagnosticsPlugin` selbst hinzu, wenn Feature `tracing-tracy` aktiv ist (`bevy_render/src/lib.rs:381-382`; `bevy/trace_tracy` aktiviert `bevy_render?/tracing-tracy`). Der Skill empfiehlt `--features bevy/trace_tracy` und zugleich `app.add_plugins((DefaultPlugins, RenderDiagnosticsPlugin))` (`SKILL.md:78`, `render-and-platform-profiling.md:9`). `Plugin::is_unique` ist standardmäßig `true`, ein zweites Hinzufügen sollte also panicen. Skill sollte `#[cfg]`/Hinweis ergänzen. Hinweis auch: `FrameTimeDiagnosticsPlugin` und `EntityCountDiagnosticsPlugin` sind Structs mit Feldern, in Code `::default()` verwenden (der Skill nennt sie nur beim Namen).

Konflikte: Steam-Deck- und WebGPU-Budgets (`SKILL.md:16-17, 109-116`, `budgets-and-telemetry.md:16-19`, Backend-Matrix) und Voxel-Beispielnamen (`voxel/...`, `bevy-voxel-runtime`-Link) sind projektfremd. Beim Übernehmen WebGPU/Browser-Zeilen und Voxel-Verweise kürzen. Sachlich unschädlich.

### bevy-cameras (14 geprüft, 1 klein falsch)

Bestätigt: `RenderTarget` als eigene Komponente mit `RenderTarget::Image(handle.into())` (Beispiel `3d/render_to_texture.rs:78`), `Camera { order: -1 }`, `ImageRenderTarget::scale_factor: f32` (`bevy_camera/src/camera.rs:990`), `Image::new_fill`-Signatur, `Projection::Orthographic(OrthographicProjection { scale, ..default_3d() })`, `AmbientLight` als Camera-Komponente, `GlobalAmbientLight` als Resource, `bevy::camera_controller::free_camera::{FreeCamera, FreeCameraPlugin}`, `pan_camera::{PanCamera, PanCameraPlugin}` (Beispiele), Features `free_camera`, `pan_camera`. Rapier-Sphere-Cast (`ReadRapierContext::single`, `cast_shape` mit `filter`, `QueryFilter::exclude_sensors/exclude_rigid_body`) stimmt gegen `bevy_rapier3d-0.36.0`.

Falsch:
- `SKILL.md:133`: "`bevy::light::GlobalAmbientLight` ... None are in the prelude". `GlobalAmbientLight` ist im Prelude (`bevy_light/src/lib.rs:73-79`). Der zusätzliche Import schadet nicht (Projekt macht ihn in `view.rs:10`), die Aussage ist aber falsch. `RenderTarget` ist tatsächlich nicht im Prelude.

Konflikte: `references/third-person-orbit.md` Abschnitt "Rapier sphere cast" (Z. 88-139) ist Rapier-only; Projekt hat Avian (`SpatialQuery::cast_shape` o.ä., nicht geprüft). Dieser Block muss umgeschrieben werden. Der Rest der Referenz (Exp-Smoothing, Fixed-Interpolation, `TransformSystems::Propagate`) ist engine-neutral und passt zur Projektarchitektur (`FixedLast`-Pose, Interpolation in `view.rs`). Tote Links: input-actions, physics, pbr-materials, a11y.

### bevy-migration-0-18-to-0-19 (~35 geprüft, 3 falsch)

Gegen die offiziellen Guides unter `bevy-0.19.1/_release-content/migration-guides/` und den Quellcode geprüft; bestätigt: `init_non_send`/`insert_non_send` (deprecated Aliase), `SceneRoot`→`WorldAssetRoot` und die ganze Typ-Umbenennungstabelle, `Replace`→`Discard`, `ExecutorKind`→`set_executor`, `shadows_enabled`→`shadow_maps_enabled`/`contact_shadows_enabled`, `ContactShadows` (`bevy_pbr/src/contact_shadows.rs`), `Atmosphere` in `bevy::light` als eigene Entity, `AssetServer::load_builder` und `LoadContext::load_builder` (`with_settings`, `override_unapproved`, `load_value`, `load_untyped`), `Reader::seekable -> Result<&mut dyn SeekableReader, ReaderNotSeekableError>`, `InputFocus::{get, set(entity, FocusCause), clear}` (die 0.19.1-Korrektur im Skill stimmt, `lib.rs:139`), `TextLayout::{justify, linebreak, no_wrap}`, `FontSize::{Px, Vw, Vh, VMin, VMax, Rem}` + `RemSize`, `audio`/`ui` nicht mehr in `2d`/`3d` impliziert (`Cargo.toml: 2d, 3d` enthalten weder `ui` noch `audio`, `default` schon), `FeathersCorePlugin`.

Falsch:
1. `references/text-and-fonts.md:30` und `:56`: `FontSource::family("Fira Sans")`. Es gibt keine Funktion `family` (nur Variante `FontSource::Family(SmolStr)` und `From<&str>`; `bevy_text/src/text.rs:282-345`). Richtig `FontSource::Family("Fira Sans".into())` oder `"Fira Sans".into()`. Der Fehler stammt aus dem offiziellen Guide `TextFont_font_and_font_size_changes.md`.
2. `references/rendering-assets-and-api.md:15-16`: `ManageViews` "split into `CreateViews`, `PrepareViews`, and `PrepareViewAttachments`". Richtig: `CreateViews`, `Specialize`, `PrepareViews` (Guide `manage-views.md`).
3. `references/resources-as-components.md:33-39`: "inserting a resource type as a component onto an entity can despawn other copies". Real wird der neu eingefügte Wert entfernt, nichts despawnt (`resource.rs`, `IsResource::on_insert`). Zusätzlich fehlt der Hinweis auf `Resource<Mutability = Mutable>`-Bound für generische `ResMut<R>` (Guide), und `IsResource` hat keinen Prelude-Eintrag (Import `bevy::ecs::resource::IsResource`).

Nicht prüfbar: glTF-Label `#Material0` → `GltfMaterial` mit `/std` (Typ `GltfMaterial` existiert, `bevy_gltf/src/lib.rs:163`, Suffix nicht gefunden).
Themenfremd: `sudo apt install libfontconfig1-dev` (text-and-fonts.md:37), "web/platform builds" in Checkliste (SKILL.md:28-29, 138-139). Tote Links: cargo-features, ui, assets.

### bevy-capture (11 geprüft, 0 falsch, nicht voll verifiziert)

`bevy_capture 0.6.0` hängt laut crates.io von `bevy ^0.19.0` ab, Features `gif`, `mp4_openh264`, `mp4_ffmpeg_cli`, `mp4_ffmpeg_cli_pipe`; docs.rs bestätigt `CapturePlugin`, `CaptureBundle`, `Capture::{start, stop, is_capturing}`, `RenderTargetHeadless::target_headless(u32, u32, &mut Assets<Image>)`, Module `frames`, `gif`, `mp4_*`. Konstruktor-Signaturen der Encoder (`Mp4Openh264Encoder::new(file, u16, u16)`) und `CaptureBundle::default` nicht verifizierbar (Crate nicht in Registry). Bevy-Seite des Beispiels bestätigt: `ScheduleRunnerPlugin { run_mode: RunMode::Loop { wait: None } }` (`bevy_app/src/schedule_runner.rs`), `RenderPlugin::synchronous_pipeline_compilation` (`bevy_render/src/lib.rs:135`), `DefaultPlugins.build().disable::<WinitPlugin>()`.
Für das Projekt niedrige Priorität: Screenshots laufen schon über `Screenshot::primary_window()` + `--hidden`; Video/MP4 wird nicht gebraucht, OpenH264 bringt Lizenz-/Patentfragen (Skill erwähnt das selbst). WASM-Hinweise (Z. 130-131, 137) und Links (cargo-features, wasm-webgpu) kürzen. Wenn aufgenommen, dann ohne externen ffmpeg-Pfad (Org-/Sicherheitsaspekt: externer Prozess).

### bevy-porting (~60 geprüft, 13 falsch) — drop

83 Codeblöcke in 11 Referenzen, die meisten API-lastig und teils veraltet. Falsch (Datei:Zeile, Befund, korrekt):
1. `godot.md:63`: `commands.trigger_targets(e, entity)`: existiert nicht mehr (`grep fn trigger_targets` in bevy_ecs ohne Treffer). Richtig: `EntityEvent` mit `entity`-Feld und `commands.trigger(e)`.
2. `unity-audio.md:386, 402, 403, 423`: `Volume::new(x)`. `Volume` ist ein Enum (`Linear(f32)`, `Decibels(..)`, `bevy_audio/src/volume.rs:36`); `Volume::new` existiert nicht. Richtig `Volume::Linear(x)`.
3. `unity-audio.md:375`: `SpatialScale::new(0.01)` als Komponente im `spawn`-Tupel. `SpatialScale` ist keine Komponente (`audio.rs:204-205`), gehört in `PlaybackSettings { spatial_scale: Some(..), .. }`.
4. `unity-audio.md:397-404, 414-425`: `Query<&AudioSink>` + `sink.set_volume(..)`. `AudioSinkPlayback::set_volume(&mut self, ..)` (`sinks.rs:26`) braucht `&mut AudioSink`.
5. `unity-audio.md:434`: "ohne `SpatialListener` fällt Bevy auf die Primary-Camera zurück". Real: Default-Ohrpositionen im Ursprung (`audio_output.rs:55-65`).
6. `unity-animation.md:70`: `graph.get_mut(node_index).weight = ..`. `AnimationGraph::get_mut` liefert `Option<&mut AnimationGraphNode>` (`graph.rs:645`).
7. `unity-assets.md:249`: Cargo-Feature `"zstd"`. Existiert nicht in bevy 0.19.1 (nur `zstd_c`, `zstd_rust`; `3d_api` enthält `zstd_rust` schon). Cargo bricht ab.
8. `unity-ui.md:509`: `LayoutAlgorithm::Flex`. Existiert nicht, richtig `Display::Flex` (`bevy_ui/src/ui_node.rs:1154`).
9. `cocos.md:232`: `TargetCamera`. Heißt `UiTargetCamera` (`ui_node.rs:2938`).
10. `gamemaker.md:105`: `vel.linvel.y`. Rapier-0.36-`Velocity` hat `linear`/`angular` (`bevy_rapier3d/src/dynamics/rigid_body.rs:88-96`); der eigene `bevy-physics`-Skill sagt es richtig.
11. `defold.md:411`: `RapierContext::cast_ray` als Typ-Methode. Ist seit 0.36 über `ReadRapierContext::single()` zu holen (nur geprüft, dass die alte Form kein SystemParam-Aufruf ist).
12. `unity-build.md:129-132` und mehrere `Cargo`-Zeilen sind Web/WASM-orientiert (widerspricht "kein Web-Export").
13. `unity-audio.md:436`: "`PlaybackSettings::DESPAWN` ... Handle nur in `AudioPlayer`": harmlos; hier nur als Beleg, dass das Spatial-Gotcha "ohne Transform silent" ungenau ist (ohne `GlobalTransform` warnt Bevy und nimmt Nullpunkt, `audio_output.rs:126-130`).

Bestätigt (Auswahl): Gamepad-API (`Gamepad::get/just_pressed`, `GamepadConnectionEvent` als Message mit `gamepad`/`connection`), `Touches::iter_just_pressed`, `Window::cursor_position`, `AnimationTransitions::play`, `AnimationEvent`-Derive und `trigger().target`, `AnimationTargetId::from_name`, `ImageNode`, `TextShadow { offset, color }`, `FontSize::Px`, `CollisionEvent::Started(a, b, _)`, `EasingCurve::new(..).sample_clamped`, `ChildOf`, `PlaybackSettings::{ONCE, LOOP, DESPAWN}`.

Warum drop: Das Projekt portiert nichts von Unity/Unreal/Roblox/Phaser/Flash/Defold/GameMaker/Cocos. Rund 20 Prozent der geprüften Beispiele sind falsch, der Rest müsste weiter gepflegt werden. Einzig `godot.md` ist für die Godot-Spikes relevant (nach Fix von Nr. 1 und mit Verweis auf Avian statt Rapier). Empfehlung: nur `references/godot.md` + `scripts/godot/tscn_inventory.py` (stdlib, liest nur `.tscn`, schreibt nur nach `--out`) als Einzeldatei übernehmen, nicht den Skill. Sonst eigenes kurzes Godot→Bevy-Mapping schreiben. Die Skript-Prüfung ergab keine Auffälligkeiten.

### bevy-physics: Ausschluss berechtigt

`SKILL.md` und alle fünf References sind Rapier-spezifisch (`RapierPhysicsPlugin`, `bevy_rapier3d 0.36`, `ReadRapierContext`, `Velocity { linear, angular }`, `PhysicsSet::SyncBackend/Writeback`). Avian kommt nur an zwei Stellen vor (`SKILL.md:23-26` als "Alternative", `setup-and-scheduling.md:8` als Tabellenzeile + Link). Projekt nutzt Avian 0.7 mit eigenen Konzepten (`PhysicsPlugins`, `PhysicsSystems::First/Last`, `PhysicsTransformConfig`, `Position`/`Rotation`, f64). Mit dem Skill würde ein Agent Rapier-API vorschlagen. Nicht übernehmen. Wenn Physik-Wissen gewünscht ist: eigener Avian-Skill aus `avian3d-0.7.0`.

## Widersprüche zum Projekt (gesammelt)

- Avian statt Rapier: `bevy-rendering` (physics-boundary.md, Audit-Skript), `bevy-cameras` (third-person-orbit.md Sphere-Cast), See-also in core-concepts, testing, rendering, cameras, Router Regel 10.
- Kein Web-Export: Web/WASM/WebGPU-Aussagen in `bevy-diagnostics-profiling`, `bevy-capture`, `bevy-rendering`, `bevy-migration-0-18-to-0-19` (Checkliste), `bevy-porting`.
- Simple-Look: Deferred-Rendering-Abschnitt in `bevy-rendering` nur als Hinweis, nicht als Standard.
- Fremdspiel-Inhalte: keine Inhalte, Namen oder Assets anderer Spiele gefunden; `bevy-porting` nennt nur Engines. Kein Verstoß.
- Tote Links nach Teilübernahme (zeigen auf nicht übernommene Skills): bevy-input-actions, bevy-physics, bevy-cargo-features, bevy-wasm-webgpu, bevy-pbr-materials, bevy-vfx, bevy-a11y, bevy-ui, bevy-assets, bevy-voxel-runtime, bevy-migration-0-17-to-0-18, bevy-states. Beim Übernehmen die See-also-Blöcke bereinigen oder `scripts/lint-skills.py` anpassen.

## Empfohlene Reihenfolge der Korrekturen

1. ecs-systems: Klammern bei Run-Conditions, `OnTransition { exited, entered }`, `LogLevel`-Import, "Freeable states" streichen.
2. rendering und migration: `PrepareViewAttachments` → `Specialize`; `FontSource::Family`.
3. ecs-components: `SetEntityEventTarget`-Pfad; Resource-Ownership-Aussage in components, Router, migration korrigieren.
4. diagnostics: Hinweis zur Doppel-Registrierung von `RenderDiagnosticsPlugin` bei `trace_tracy`.
5. Rapier/Web/Voxel-Passagen kürzen, Router und See-also anpassen.
