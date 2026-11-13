# Splicing-Guided Hybrid Homomorphic Encryption for Genomic Data (e-PHI)

![Python](https://shields.io)
![TenSEAL](https://shields.io)
![Security](https://shields.io)

## 📌 Overview
**Splicing-Guided Hybrid FHE**는 생물학적 선택적 스플라이싱(Alternative Splicing) 메커니즘에서 영감을 받아, 대규모 유전체 데이터(VCF)의 완전동형암호(FHE) 연산 병목과 저장 공간 부하를 해결하기 위해 설계된 하이브리드 프라이버시 보존 파이프라인입니다.

전체 유전체 변이를 무작정 동형암호화하는 기존 방식 대신, 임상 분석(예: 다중유전자 위험점수, PRS)에 직접 사용되는 **엑손(Exon) 영역만 추출하여 CKKS 스킴으로 암호화**하고, 나머지 **인트론(Intron) 영역은 AES-256-GCM 및 HMAC-SHA256으로 경량 분리 보관**합니다.

---

## 🏗️ System Architecture

1. **Functional Splicing (Exon/Intron Partitioning)**: GENCODE GTF 좌표를 기반으로 환자 VCF 데이터에서 분석 대상 Exon 변이와 비코딩 Intron 변이를 분리합니다.
2. **Metadata Leakage Mitigation (Padding & Shuffling)**: 
   - **Fixed-Size Bucket Padding**: 특정 질환 패널 유추 공격(Size-based Inference)을 막기 위해 고정된 슬롯 크기(`8,192`)로 더미 패딩을 적용합니다.
   - **PRNG Slot Shuffling**: 비밀키 시드 기반 순열을 통해 유전자 위치 정보를 난독화합니다.
3. **Dual-Layer Encryption & Cryptographic Binding**:
   - **Exon**: `TenSEAL (CKKS)` 동형암호화 후 클라우드 연산 서버로 전송.
   - **Intron**: `AES-256-GCM` 암호화 후 로컬/별도 스토리지 보관 및 `HMAC-SHA256` 태그로 무결성 바인딩(HIPAA 보안 규칙 충족).
4. **Privacy-Preserving Cloud Computation**: 클라우드 서버가 복호화 없이 SIMD 내적 연산을 통해 위험점수(PRS)를 산출합니다.

---

## 📊 Experimental Benchmark (1000 Genomes Project - Chr22)

- **Dataset**: 1000 Genomes Phase 3 Chromosome 22 (`1,103,547` total variants / `107,814` exon variants, 9.77%)
- **Environment**: Python 3.10, TenSEAL (CKKS, $N=16384$, Scale=$2^{40}$), AES-256-GCM

| Metric | Baseline (Full WGS FHE) | Proposed (Splicing Hybrid FHE) | Improvement |
| :--- | :---: | :---: | :---: |
| **FHE Ciphertext Chunks** | 135 chunks | **14 chunks** | **89.6% Reduction** |
| **Total Encrypted Size** | 84.70 MB | **19.23 MB** | **77.30% Reduction** |
| **Client Encryption Time** | 1.9260 sec | **0.2258 sec** | **8.53x Faster** |
| **Cloud Evaluation Time (PRS)** | 6.8476 sec | **0.6499 sec** | **10.54x Faster** |
| **Computation Accuracy** | Baseline | **99.9998% Match** | Abs. Error $\approx 1.9 \times 10^{-4}$ |

---

## 🚀 Quick Start

```bash
# 1. Install dependencies
pip install tenseal cryptography numpy

# 2. Download public datasets (1000 Genomes Chr22 & GENCODE GTF)
wget https://ucsc.edu
wget https://ebi.ac.uk

# 3. Run benchmark
python benchmark.py
```
