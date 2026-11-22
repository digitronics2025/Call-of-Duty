# Call of Duty agent contract

This is a Unity 6 LTS, URP, offline Windows single-player game. Read the
[engineering context](engineering-context.md) for design history and current
project context. Check the relevant [system guide](systems/README.md) before
changing a subsystem. When code and an active rule conflict, update this
contract first.

## Active design and engineering rules

- Keep the game offline. Do not add networking. Use the new Input System only.
- `GameMode` has exactly `Run` and `Sandbox`; campaign is a separate content
  setting. Preserve existing save compatibility unless a migration is explicitly
  requested and reviewed.
- Store tunable values in ScriptableObject assets under
  `Assets/_Project/Data/`. Do not add tunable public fields to MonoBehaviours or
  hard-code gameplay numbers in scripts.
- Do not call `Find`, `GetComponent`, `Camera.main`, `Instantiate` or `Destroy`
  from `Update`, `FixedUpdate` or `LateUpdate`. Pool everything that spawns.
  Avoid per-frame allocations and mutable static state.
- Keep physics in `FixedUpdate`, input in `Update`, and camera work in
  `LateUpdate`. First-party C# files start with `#nullable enable` and compile
  without warnings.
- Generated scenes, prefabs and tuning assets come from the repository builders.
  Edit their source and rerun the builder; do not hand-edit generated output.
- Do not edit bought assets in `Assets/ThirdParty/`. Preserve licenses, GUIDs,
  `.meta` files, Git LFS coverage, project identity and signing material.
- Keep first-party textures within the 1024 limit (2048 only for weapons and
  hands). The target laptop has 4 GB VRAM. See the subsystem guides for budgets.
- Save data is versioned JSON, written through a temporary file with a backup.
  Preserve this recovery path and existing player saves.

## Verification

- Run `node Tools/typecheck.mjs` and `node Tools/check.mjs` for code changes.
- For changes that affect game behavior, run both Unity suites headlessly:

  ```text
  Unity.exe -batchmode -runTests -projectPath . -testPlatform EditMode -testResults Logs/tests-editmode.xml
  Unity.exe -batchmode -runTests -projectPath . -testPlatform PlayMode -testResults Logs/tests-playmode.xml
  ```

  A compiled assembly without its relevant tests is unverified.
- For audio routing changes, run
  `Unity.exe -batchmode -quit -projectPath . -executeMethod CoD.EditorTools.AudioBuilder.VerifyMixerHeadless`.
- For scenes, Build Settings, saves or `UNITY_EDITOR` changes, close Unity and
  run `node Tools/verify-build.mjs`; it builds and boots the Windows player.
- Update the code-verified guide in `docs/systems/` in the same change as its
  subsystem. Do not weaken or remove a guard to make a check pass.

## Data, tools and ownership

- Read the [MyVault agent guide](https://myvault.digitronics-electro.workers.dev/downloads/myvault-agent-guide.txt)
  before using credentials. Keep secrets out of Git, logs and replies.
- `.claude/settings.json` pre-approves the Cloudflare tools `search`, `docs`,
  `whoami` and `execute`; this does not authorize live changes. Get explicit
  user approval before changing any live Cloudflare resource.
- Advice and investigation are read-only. Reviews lead with verified defects;
  improvement reviews also consider relevant workflows and failure recovery.
  Implement only when asked, and keep gameplay, art and assets out of an
  instruction-only change.
- Use a scoped pull request and the repository's checks for changes. Preserve
  unrelated work. Merge only after required checks and human approvals pass.
