# rEFInd 0.14.2 — monitor-aware / headless build

Repozytorium automatycznie buduje zmodyfikowany rEFInd 0.14.2 dla UEFI x86-64.
Nie zawiera binarnego bootloadera ani kopii źródeł upstream: GitHub Actions pobiera
źródła z oficjalnej strony projektu na SourceForge, sprawdza SHA-256, stosuje patch
i kompiluje aplikację przez GNU-EFI.

## Co robi workflow

1. Pobiera `refind-src-0.14.2.tar.gz` z SourceForge.
2. Weryfikuje SHA-256 archiwum: `f7d93ce80da76b86c567281ea225b6a87907ce86ff77233c9357a522c115c8f0`.
3. Sprawdza wersję źródeł i nakłada `patches/refind-0.14.2-headless.patch` bez fuzz.
4. Buduje `refind/refind_x64.efi` na Ubuntu 22.04 z GNU-EFI.
5. Pakuje plik jako `EFI/BOOT/BOOTX64.EFI` i publikuje artefakt GitHub Actions.

## Jak uruchomić kompilację

1. Utwórz nowe, puste repozytorium na GitHubie (np. `refind-headless`).
2. Rozpakuj zawartość tego pakietu ZIP.
3. Wgraj wszystkie pliki i foldery do repozytorium, zachowując strukturę, w szczególności `.github/workflows/build.yml`.
4. Otwórz repozytorium na GitHubie i przejdź do zakładki **Actions**.
5. Wybierz workflow **Build rEFInd headless (0.14.2)**.
6. Kliknij **Run workflow** i zatwierdź przyciskiem **Run workflow**.
7. Po zakończeniu zadania otwórz jego stronę i pobierz artefakt `BOOTX64-EFI-headless-...` z sekcji **Artifacts**.

Workflow uruchomi się też automatycznie po zmianie patcha, README lub pliku workflow na gałęzi `main`/`master`.

## Wynik

W pobranym artefakcie znajdziesz:

- `EFI/BOOT/BOOTX64.EFI` — skompilowany program UEFI x86-64;
- `refind.conf.example` — upstreamowy przykładowy plik konfiguracji (nie zastępuje Twojego własnego `refind.conf`);
- `BUILD-INFO.txt` i `SHA256SUMS.txt` — informacje o kompilacji i sumy kontrolne.

**Ważne:** workflow buduje tylko `BOOTX64.EFI`. Nie kopiuje automatycznie motywu, ikon, sterowników EFI ani Twojego `refind.conf` na partycję EFI. Zachowaj obecny układ katalogów rEFInd i najpierw testuj nowy plik z USB lub osobnego wpisu UEFI. Nie nadpisuj jedynego działającego bootloadera.

## Założenia i ograniczenia poprawki

- Wczesna detekcja używa protokołów EDID Active/Discovered. EDID nie jest gwarantowanym czujnikiem fizycznego podłączenia monitora; wynik zależy od firmware, GPU, adapterów i KVM.
- Ścieżka headless jest przygotowana dla konkretnego komputera: GPT GUID partycji TrueNAS `e111363b-0d34-4d54-b45a-c592e9be7e2c` i loader `\\EFI\\debian\\grubx64.efi`.
- `headless_default_selection` akceptuje `TrueNAS`; cel partycji i ścieżka GRUB są stałe w patchu.
- Jeżeli GRUB zwróci sterowanie lub nie da się go uruchomić, rEFInd nie przechodzi do GUI. Zwraca błąd do firmware, aby nie ryzykować zawieszenia na ekranie bez monitora.
- Sam poprawny build nie potwierdza zachowania na fizycznym komputerze. Konieczne są osobne testy z monitorem i bez monitora.

## Zmiana celu TrueNAS

Jeśli zmienisz GUID partycji lub ścieżkę GRUB, zmień wartości w `patches/refind-0.14.2-headless.patch`, a następnie uruchom workflow ponownie. Nie edytuj wyłącznie pliku konfiguracyjnego — obecna implementacja ma cel TrueNAS wbudowany w kod.

## Licencja

rEFInd jest dystrybuowany na warunkach GNU GPL v3 lub późniejszej. Patch w tym repozytorium modyfikuje kod rEFInd i należy go używać zgodnie z licencją projektu upstream.
