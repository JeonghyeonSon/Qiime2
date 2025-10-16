## Using VCL computer in NCSU

# 1. Miniforge 설치 파일 다운로드
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"

# 2. 설치 실행
bash Miniforge3-Linux-x86_64.sh -b

# 3. Conda 초기화 및 적용
~/miniforge3/bin/conda init
source ~/.bashrc

# 4. search giime 2 download through google, and find install qiime2 within a conda environment...
이런... 곳에 접속해서 https://docs.qiime2.org/2024.10/install/native/

conda env create -n qiime2-amplicon-2024.10 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml

# 5. 내 컴퓨터 파일을 vcl로 올리기...
scp -r "/mnt/d/OneDrive/qiime/yesid_raw data/zr25695.rawdata.250827" json7@vclvm178-152.vcl.ncsu.edu:~/
# 질문나오면 yes라 하고.. 내 portal password 도 넣어주면 파일을 옮겨줌


