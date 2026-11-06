# Known install issues

Both issues below hit every fresh install of the RPent environment
([README → RPent](../README.md#rpent)); neither is specific to a GPU vendor.
Both were reported in
[#3](https://github.com/xinxinliu-ai/AgenticRoboRL/issues/3).

## 1. sam3 0.1.4 ships without its BPE vocabulary

**Symptom:** the sam3 server fails at startup with a `FileNotFoundError`
pointing at `assets/bpe_simple_vocab_16e6.txt.gz`.

**Cause:** the PyPI package loads this CLIP BPE vocabulary at startup but does
not include it in the wheel.

**Fix:** download the identical file from
[openai/CLIP](https://github.com/openai/CLIP) into the installed package:

```bash
BPE=$($RPENT_ENV/bin/python -c "import os, sam3; print(os.path.join(os.path.dirname(os.path.dirname(sam3.__file__)), 'assets', 'bpe_simple_vocab_16e6.txt.gz'))")
mkdir -p "$(dirname "$BPE")" && curl -fsSL -o "$BPE" https://github.com/openai/CLIP/raw/main/clip/bpe_simple_vocab_16e6.txt.gz
```

## 2. liberopro's floor-mounted robots crash robosuite 1.5.2

**Symptom:** environment creation fails for every `libero_object_*` cell, with
robosuite unable to look up `None` in its `BASE_MAPPING`.

**Cause:** the robots' `default_base` returns `None` — robosuite 1.4's way of
saying "no mount". robosuite 1.5.2 looks the value up in `BASE_MAPPING`, whose
name for "no mount" is `"NullMount"`.

**Fix:** patch the installed package:

```bash
$RPENT_ENV/bin/python - <<'EOF'
import glob, os, re
import liberopro.liberopro.envs.robots as pkg
for f in sorted(glob.glob(os.path.join(os.path.dirname(pkg.__file__), "*.py"))):
    src = open(f).read()
    new = re.sub(r"(def default_base\(self\):\s*\n\s*return )None\b", r'\1"NullMount"', src)
    if new != src:
        open(f, "w").write(new)
        print("patched:", os.path.basename(f))
EOF
```

> [!NOTE]
> Reinstalling the RPent extras restores liberopro's original files and
> silently undoes this edit — re-run the snippet above after any reinstall.
