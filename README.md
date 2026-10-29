# Research-Work

## CER4

The `CER4/` folder contains the preprocessing, normalization and final
normalized-window files for the CER4 work.

```
CER4/
  1_preprocessing/             cer, lcl, sgcc and etd2022 daily files
  2_normalization/
  3_final_common_normalized/   final 30-day windows (large files, split - see below)
  Documents/
```

### Large files in `3_final_common_normalized/`

GitHub rejects files over 100 MB, so the ten `final_windows_30d_seed_*.xlsx`
files are stored as consecutive 90 MiB pieces named
`<original name>.part001`, `.part002`, ... Join the pieces back together to get
the original `.xlsx` files.

**Linux / macOS / Git Bash**

```bash
cd CER4/3_final_common_normalized
for f in *.xlsx.part001; do b="${f%.part001}"; cat "$b".part* > "$b"; done
sha256sum -c SHA256SUMS.txt
```

**Windows PowerShell**

```powershell
cd CER4\3_final_common_normalized
Get-ChildItem *.part001 | ForEach-Object {
    $b = $_.Name -replace '\.part001$', ''
    $out = [IO.File]::Create((Join-Path $PWD $b))
    Get-ChildItem "$b.part*" | Sort-Object Name | ForEach-Object {
        $bytes = [IO.File]::ReadAllBytes($_.FullName)
        $out.Write($bytes, 0, $bytes.Length)
    }
    $out.Close()
}
Get-Content SHA256SUMS.txt | ForEach-Object {
    $hash, $name = $_ -split '\s+', 2
    "{0}  {1}" -f ((Get-FileHash $name -Algorithm SHA256).Hash.ToLower() -eq $hash), $name
}
```

`SHA256SUMS.txt` lists the checksum of each original file, so you can confirm the
rejoined files are identical.
