---
title: NFS 서비스
source: Notion
created: 2026-03-31
---

# NFS 서비스

![[attachments/image 37.png]]

### 1. 개요

- **NFS (Network File System)**: 네트워크 상의 다른 컴퓨터와 파일을 공유할 수 있게 해주는 프로토콜입니다. 주로 Linux/Unix 환경에서 사용됩니다.

### 2. 실습 환경

- OS: Rocky Linux 8
- 실습 목표:

  **NFS** 설정하여 Linux 간 파일 공유

---

### 3. **NFS 설정 실습**

### 3.1. NFS 서버 설치

1. **NFS 서버 설치**
    
    ```bash
    sudo dnf install -y nfs-utils
    
    ```
    
2. **NFS 서비스 시작 및 부팅 시 자동 시작 설정**
    
    ```bash
    sudo systemctl enable --now nfs-server
    
    ```
    
3. **방화벽 설정 (포트 2049 허용)**
    
    ```bash
    sudo firewall-cmd --permanent --add-service=nfs
    sudo firewall-cmd --reload
    
    ```
    
4. **NFS 공유 디렉토리 생성**
    
    예: `/srv/nfs_share` 디렉토리 생성
    
    ```bash
    sudo mkdir -p /srv/nfs_share
    sudo chown nfsnobody:nfsnobody /srv/nfs_share
    sudo chmod 755 /srv/nfs_share
    ```
    
5. **NFS 공유 설정**
    
    `/etc/exports` 파일에 공유할 디렉토리 추가
    
    ```bash
    echo "/srv/nfs_share *(rw,sync,no_root_squash)" | sudo tee -a /etc/exports
    ```
    
6. **NFS 설정 적용**
    
    ```bash
    sudo exportfs -r
    ```
    

### 3.2. 클라이언트에서 NFS 공유 마운트

1. **NFS 클라이언트 설치**
    
    ```bash
    sudo dnf install -y nfs-utils
    ```
    
2. **NFS 공유 디렉토리 마운트**
    
    ```bash
    sudo mount -t nfs <서버_IP>:/srv/nfs_share /mnt
    ```
    
3. **마운트된 공유 디렉토리 확인**
    
    ```bash
    df -h
    ```
    
4. **영구 마운트 설정 (옵션)**
    
    `/etc/fstab` 파일에 NFS 공유 디렉토리 추가
    
    ```bash
    <서버_IP>:/srv/nfs_share /mnt nfs defaults 0 0
    ```
    

### 3.3. NFS 서비스 확인

1. **서버에서 NFS 서비스 상태 확인**
    
    ```bash
    sudo systemctl status nfs-server
    ```
    
2. **클라이언트에서 NFS 공유 확인**
    
    ```bash
    showmount -e <서버_IP>
    ```