@echo off
title Otomatisasi Git Super Bersih
cls

echo =======================================================
echo                 GIT AUTOMATION CLEANER
echo =======================================================

:: 1. DETEKSI GIT
echo [*] Mencari Git di komputer Anda...
where git >nul 2>nul
if %errorlevel% equ 0 (
    set "GIT_CMD=git"
    echo [OK] Git Sistem ditemukan.
    goto :INPUT_LINK
)

if exist "D:\GitPortable\bin\git.exe" (
    set "GIT_CMD=D:\GitPortable\bin\git.exe"
    echo [OK] Git Portable ditemukan di D:\GitPortable
    goto :INPUT_LINK
)

if exist "C:\GitPortable\bin\git.exe" (
    set "GIT_CMD=C:\GitPortable\bin\git.exe"
    echo [OK] Git Portable ditemukan di C:\GitPortable
    goto :INPUT_LINK
)

echo [EROR] Git tidak ditemukan!
echo Pastikan folder GitPortable Anda berada di D:\GitPortable atau C:\GitPortable
pause
goto :PEMBERSIHAN_AKHIR

:INPUT_LINK
echo =======================================================
echo 2. INPUT LINK GITHUB
echo =======================================================
set /p "LINK_DOWNLOAD=Masukkan Link GitHub untuk DOWNLOAD (V1/Source): "
set /p "LINK_UPLOAD=Masukkan Link GitHub Baru untuk UPLOAD (V2): "

echo =======================================================
echo 3. PROSES DOWNLOAD (CLONE / FETCH)
echo =======================================================
echo [*] Mengunduh seluruh file proyek...
"%GIT_CMD%" init
"%GIT_CMD%" remote add origin_old "%LINK_DOWNLOAD%"
"%GIT_CMD%" fetch origin_old
"%GIT_CMD%" checkout -b main origin_old/main --force
"%GIT_CMD%" branch -m main

if %errorlevel% neq 0 (
    echo [EROR] Gagal download. Periksa koneksi atau link Anda.
    goto :PEMBERSIHAN_AKHIR
)
echo [OK] Download selesai! Silakan masukkan/ubah file V2 Anda di folder ini jika diperlukan.
echo Tekan tombol apa saja jika sudah siap untuk melakukan UPLOAD ke akun baru...
pause >nul

echo =======================================================
echo 4. PROSES UPLOAD (PUSH) KE AKUN BARU
echo =======================================================
echo [*] Menyiapkan sambungan ke akun baru...
"%GIT_CMD%" remote remove origin >nul 2>nul
"%GIT_CMD%" remote remove origin_old >nul 2>nul
"%GIT_CMD%" remote add origin "%LINK_UPLOAD%"

echo [*] Menambahkan file baru dan mengirim ke GitHub...
"%GIT_CMD%" add .
"%GIT_CMD%" commit -m "Upload otomatis kode V2 lewat script bat"
"%GIT_CMD%" push -u origin main --force

if %errorlevel% neq 0 (
    echo [EROR] Gagal upload! Pastikan Anda sudah login di jendela popup yang muncul.
    goto :PEMBERSIHAN_AKHIR
)
echo [OK] Proses Upload Berhasil 100%!

echo =======================================================
echo 5. MEMBERSIHKAN KREDENSIAL LOGIN PC
echo =======================================================
echo [*] Menghapus data login dari Windows Credential Manager...
cmdkey /list | findstr /I "git:https://github.com" >nul
if %errorlevel% equ 0 (
    cmdkey /delete:git:https://github.com
)
echo url=https://github.com | "%GIT_CMD%" credential-manager erase >nul 2>nul
"%GIT_CMD%" credential reject >nul 2>nul
echo [OK] Data login Anda telah dihapus total dari PC ini. Aman!

:PEMBERSIHAN_AKHIR
echo =======================================================
echo 6. PEMBERSIHAN FILE PROYEK
echo =======================================================
set /p "HAPUS_FILE=Apakah Anda ingin MENGHAPUS semua file proyek yang di-download tadi? (Y/T): "

if /I "%HAPUS_FILE%"=="Y" (
    echo [*] Sedang membersihkan folder ini secara total...
    for /f "delims=" %%i in ('dir /b /a') do (
        if not "%%i"=="%~nx0" (
            if not "%%i"=="bersih.bat" (
                if not "%%i"=="bersihkan.bat" (
                    if exist "%%i\*" (
                        rd /s /q "%%i"
                    ) else (
                        del /f /q /a "%%i"
                    )
                )
            )
        )
    )
    if exist .git rd /s /q .git
    echo [OK] Folder kembali BERSIH TOTAL!
) else (
    echo [OK] Terima kasih! File proyek tetap disimpan di PC Anda.
)

echo =======================================================
echo Selesai! Script akan menutup otomatis.
echo =======================================================
pause
exit