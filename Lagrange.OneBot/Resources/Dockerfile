##############################
# 阶段1：构建 .NET 项目
##############################
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:9.0-bookworm-slim AS dotnet-build
WORKDIR /root/build
# 如果需要针对不同架构构建，可传入 TARGETARCH 参数
ARG TARGETARCH
COPY . /root/build
# 发布 .NET 项目（请根据实际项目名称与参数调整）
RUN dotnet publish -p:DebugType="none" -a $TARGETARCH -f "net9.0" -o "/root/out" "Lagrange.OneBot"

##############################
# 阶段2：构建 Python 环境
##############################
FROM python:3.10-slim AS python-build
WORKDIR /AstrBot
COPY . /AstrBot/
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    build-essential \
    python3-dev \
    libffi-dev \
    libssl-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
# 安装 Python 依赖
RUN python -m pip install -r requirements.txt --no-cache-dir
RUN python -m pip install socksio wechatpy cryptography --no-cache-dir

##############################
# 阶段3：生成最终镜像（同时具备 .NET 运行时和 Python 运行环境）
##############################
FROM mcr.microsoft.com/dotnet/runtime:9.0-bookworm-slim AS final
WORKDIR /app

# 安装 Python（基于 .NET 运行时镜像安装 Python 环境）
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# 将 .NET 应用从 dotnet-build 阶段复制到镜像中
COPY --from=dotnet-build /root/out /app/bin

# 将 Python 应用从 python-build 阶段复制到镜像中
COPY --from=python-build /AstrBot /app/AstrBot

# 复制 docker-entrypoint.sh 并赋予执行权限
# （注意：此脚本位于 Lagrange.OneBot/Resources/ 目录下）
COPY Lagrange.OneBot/Resources/docker-entrypoint.sh /app/bin/docker-entrypoint.sh
RUN chmod +x /app/bin/docker-entrypoint.sh

# 安装 gosu（如有需要，用于切换用户等操作）
RUN apt-get update && apt-get install -y --no-install-recommends gosu \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# 暴露 Python 服务可能使用的端口（可根据实际情况调整）
EXPOSE 6185 6186

# 设置容器启动时执行的入口脚本
# 如需同时启动 Python 与 .NET 应用，请在 docker-entrypoint.sh 中添加相应的启动逻辑，
# 例如同时启动 "python /app/AstrBot/main.py" 和 .NET 应用。
ENTRYPOINT ["/app/bin/docker-entrypoint.sh"]

# 如果希望给出默认命令，可以追加 CMD 指令，例如：
# CMD ["python", "/app/AstrBot/main.py"]
