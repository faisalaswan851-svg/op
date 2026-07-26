---
name: elite-software-engineering-master
description: Panduan teknik rekayasa perangkat lunak tingkat lanjut untuk debugging, root-cause analysis, refaktor, tinjauan arsitektur, dan perubahan berdampak besar. Gunakan dokumen ini sebagai pedoman operasional, bukan sebagai checklist kaku.
---

# Elite Software Engineering Master

## Versi & Kepemilikan
- Pemilik dokumen: @faisalaswan851-svg
- Versi: 1.0.0
- Tanggal revisi: 2026-07-26
- Proses review: tinjauan berkala setiap 6 bulan atau setelah perubahan arsitektural besar.

## TL;DR (Ringkasan 5 poin)
1. Reproduksi masalah => pahami jalur eksekusi => identifikasi akar penyebab.
2. Buat perubahan paling kecil yang aman, sertakan rollback dan tes.
3. Selalu sediakan bukti verifikasi (log, test, benchmark).
4. Gunakan checklist PR/QA dan dapatkan sign-off dari pemangku kepentingan relevan.
5. Jika ada pengecualian (hotfix darurat), dokumentasikan alasan, mitigasi, dan follow-up.

## Tujuan
Panduan ini dimaksudkan untuk pekerjaan teknik yang berisiko tinggi: debugging kompleks, tinjauan arsitektur, refactor besar, perbaikan keamanan, dan penggantian modul. Fokusnya pada solusi yang:
- benar,
- aman,
- dapat diskalakan,
- mudah dipelihara,
- dan siap produksi.

## Prinsip Utama (Ringkas)
- Jangan menebak; buktikan.
- Ubah sekecil mungkin.
- Prioritaskan keselamatan dan kemampuan rollback.
- Dokumentasikan perubahan dan verifikasi.
- Pertimbangkan trade-off dan catat keputusan.

## Glossary (Definisi istilah kunci)
- Evidence: artefak yang dapat diperiksa — log, trace, test yang dapat direproduksi, snapshot, atau benchmark.
- Large / Berisiko Tinggi: perubahan yang menyentuh lebih dari satu layanan, database schema, public API, atau bagian kritikal produksi.
- Verified: perubahan yang telah melalui unit test, integration test (jika relevan), smoke test, dan review yang disyaratkan.
- Minimal safe change: perubahan terukur terkecil yang menyelesaikan akar penyebab tanpa menambah risiko signifikan.
- Regression: perilaku yang dulunya benar menjadi salah akibat perubahan baru.

## Prosedur Wajib (Step-by-step)
1. Definisikan masalah secara ringkas: apa yang salah, dimana (service/file), dampak yang terlihat.
2. Reproduksi: jika mungkin, buat langkah reproduksi di lingkungan non-produksi.
3. Pencatatan bukti: ambil logs, traces, core dump, sampel request/response, dan metrik performa sebelum/selesai.
4. Pemetaan: identifikasi modul terkait, dependensi, interaksi eksternal, dan titik integrasi.
5. Analisis akar penyebab: telusuri eksekusi, gunakan debugger, instrumentation, atau tracing.
6. Pilih solusi: tentukan perubahan terkecil yang memperbaiki akar masalah; jika tidak mungkin, rencanakan mitigasi sementara.
7. Evaluasi risiko: isi template Risk Assessment (di bawah).
8. Implementasi: buat branch fitur/bugfix, sertakan test dan dokumentasi singkat perubahan.
9. Verifikasi lokal & CI: jalankan unit/integration tests, linters, dan checks yang disyaratkan.
10. PR & Review: sertakan checklist PR, bukti verifikasi, dan dapatkan sign-off dari reviewer yang ditentukan.
11. Deploy & Monitoring: deploy ke staging lalu produksi sesuai prosedur; pantau metrik & alert.
12. Post-mortem / Follow-up: jika terjadi insiden, dokumentasikan temuan dan rencana pencegahan.

## Checklist PR (untuk disertakan di description PR)
- [ ] Deskripsi masalah dan referensi issue/bug.
- [ ] Langkah reproduksi (jika ada).
- [ ] Ringkasan perubahan yang dibuat.
- [ ] Bukti verifikasi: hasil test, log, screenshot, benchmark.
- [ ] Risk assessment terisi (lihat template).
- [ ] Tests baru ditambahkan / test existing diperbarui.
- [ ] Linter & formatting lulus.
- [ ] Peer review (N = 1 minimum, atau 2 untuk perubahan besar).
- [ ] Migration & rollback plan (jika ada schema/DB/API change).

## Template Risk Assessment (singkat)
- Perubahan: [singkat]
- Dampak (Impact): [High/Medium/Low] — jelaskan
- Kemungkinan (Likelihood): [High/Medium/Low] — jelaskan
- Mitigasi sebelum deploy: [tes, feature flag, canary, fallback]
- Rollback plan: [cara rollback & estimasi waktu]
- Owner (siapa bertanggung jawab): @username

## Peran & Tanggung Jawab
- Author: membuat PR, menulis bukti, dan menjalankan tes.
- Reviewer teknis: memeriksa keamanan, arsitektur, dan kebenaran implementasi.
- Release owner / Ops: menyetujui deployment ke produksi dan memonitor setelah release.
- Security reviewer: wajib untuk perubahan yang menyentuh autentikasi, otorisasi, atau data sensitif.

## Integrasi Tools & Praktik Otomatis
Rekomendasi minimal CI yang wajib dijalankan pada setiap PR:
- unit tests (jalankan `npm test` / `pytest` / sesuai stack),
- linters & formatter (ESLint, RuboCop, gofmt, dll),
- dependency scanning (SCA),
- static analysis / SAST untuk kode kritikal,
- minimal smoke tests di staging,
- coverage gating (opsional; tetapkan ambang jika ingin).

Contoh snippet GitHub Actions yang disarankan (ringkas):
```yaml
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up
        run: |
          # install deps
      - name: Run tests
        run: |
          # run unit tests and report
      - name: Lint
        run: |
          # run linter
```

## Kriteria “Done” (metrik minimal)
- Semua tests terkait lulus di CI.
- Tidak ada static analysis critical issues.
- Peer review diselesaikan dan sign-off diberikan.
- Rollback plan tersedia untuk produksi.
- Perubahan sudah didokumentasikan di changelog (atau PR description) bila perlu.

## Contoh singkat (bug kecil)
Kasus: API mengembalikan 500 pada endpoint /users ketika header X kosong.
Langkah singkat:
- Reproduksi lokal dengan curl.
- Periksa logs -> NullPointer pada parsing header.
- Perbaikan: tambahkan pemeriksaan null sebelum parsing dan unit test untuk header kosong.
- PR: sertakan test, log error sample, dan checklist PR terisi.

## Contoh perubahan berisiko (schema DB)
- Buat migration script idempotent.
- Jalankan migration di staging dan verifikasi integritas data.
- Gunakan feature flag untuk mengaktifkan perilaku baru.
- Pastikan rollback: backup DB & script rollback tersedia.

## Exceptions (kapan boleh melanggar aturan)
Hotfix produksi darurat:
- Boleh dilakukan jika ada insiden kritikal berdampak besar dan mitigasi sementara diperlukan.
- Persyaratan: tindakan minimal, catat alasan, segera buat post-release PR untuk perbaikan permanen, lakukan post-mortem.
- Harus ada notifikasi ke tim dan sign-off retroaktif dari arsitek atau owner.

## Verifikasi & Audit
Setiap perubahan besar harus menyertakan bukti: tes otomatis, benchmark (jika performa terdampak), dan logs. Simpan artefak verifikasi di PR (attachment atau link ke job CI).

## Delivery & Dokumentasi
Akhiri pekerjaan dengan ringkasan di PR: apa yang diubah, kenapa, bagaimana diverifikasi, siapa yang memeriksa, dan risiko tersisa. Jika perlu, perbarui dokumentasi pengguna (CHANGELOG/README).

## Failure Conditions (jangan lanjut jika)
- Penyebab tidak jelas atau hanya asumsi.
- Tidak ada cara reproducible untuk menguji perbaikan.
- Tidak tersedia rollback/mitigasi untuk perubahan berisiko.
- Verifikasi tidak meyakinkan.

## Verification Checklist (sebelum merge)
- [ ] Penyebab akar sudah diidentifikasi dan didokumentasikan.
- [ ] Perubahan minimal dan terverifikasi.
- [ ] Semua item pada PR checklist terpenuhi.
- [ ] Risk assessment disetujui.
- [ ] Dokumentasi dan changelog ter-update bila perlu.

## Lain-lain
- Tambahkan link ke CONTRIBUTING.md dan template PR di repo agar panduan ini mudah dipakai.
- Simpan catatan versi dokumen di header. Untuk perubahan substansial, tingkatkan versi dan catat ringkasan perubahan.

---
GitHub reference: https://github.com/faisalaswan851-svg/op
