
<h1> Servidor de Streaming com Nginx, RTMP e FFMPEG </h1>

<div style="text-align: justify;">
Nginx, RTMP e ffmpeg são três tecnologias poderosas que podem ser usadas juntas para transmitir conteúdo de vídeo e áudio em tempo real para o público. O nginx é um servidor web de alto desempenho que oferece diversas vantagens para streaming, como baixa latência e alta escalabilidade. O RTMP é um protocolo de rede que foi projetado para minimizar o atraso entre a captura e a reprodução do vídeo, o que o torna ideal para transmissões ao vivo. O ffmpeg é um software que permite preparar o conteúdo para streaming, como converter o formato de vídeo e codificá-lo para bitrates diferentes.

A combinação de nginx, RTMP e ffmpeg oferece uma solução completa para streaming de mídia em tempo real, ideal para empresas e indivíduos que desejam transmitir eventos ao vivo, webinars, aulas online, jogos e muito mais.
</div>

<h2> Exemplo de configuração para transmissão de 3 câmeras com o protocolo RTSP </h2>

<h3> Instalando e configurando o nginx e rtmp </h3>

<h4> 1 - Instalar os pacotes nginx e libnginx-mod-rtmp  </h4>

```
sudo apt install nginx libnginx-mod-rtmp 
```

<h4> 2 - Configurar o nginx.conf </h4>

- Editando o arquivo nginx.conf
```
sudo vim /etc/nginx/nginx.conf
```
- Adicionando a seguinte configuração no final do arquivo
```
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live1 {
            live on;
            record off;

            hls on;
            hls_path /var/www/html/stream/live1/hls;
            hls_fragment 3;
            hls_playlist_length 60;

            dash on;
            dash_path /var/www/html/stream/live1/dash;
        }

        application live2 {
            live on;
            record off;

            hls on;
            hls_path /var/www/html/stream/live2/hls;
            hls_fragment 3;
            hls_playlist_length 60;

            dash on;
            dash_path /var/www/html/stream/live2/dash;
        }

        application live3 {
            live on;
            record off;

            hls on;
            hls_path /var/www/html/stream/live3/hls;
            hls_fragment 3;
            hls_playlist_length 60;

            dash on;
            dash_path /var/www/html/stream/live3/dash;
        }

    }
}
```

<h4> 3 - Criar e configurar o arquivo live </h4>

- Criando o arquivo live no diretório /etc/nginx/sites-available
```
sudo touch /etc/nginx/sites-available/live
```
- Configurando o arquivo live
```
sudo vim /etc/nginx/sites-available/live
```
- Adicionando a seguinte configuração
``` 
server {
    listen 8080;
    server_name _;

    location / {
        add_header Access-Control-Allow-Origin *;
        root /var/www/html/stream;
    }

}

types {
    application/dash+xml mpd;
}
```
- Criando um link simbólico do arquivo live no diretório /etc/nginx/sites-enabled

```
sudo ln -s /etc/nginx/sites-available/live /etc/nginx/sites-enabled/live
```

<h4> 4 - Criar e configurar a estrutura de diretório </h4>

- Criando a pasta stream no diretório /var/www/html para armazenar os arquivos de transmissões
```
sudo mkdir /var/www/html/stream
```
- Criando as pastas live no diretório /var/www/html/stream
```
sudo mkdir /var/www/html/stream/live1
sudo mkdir /var/www/html/stream/live2
sudo mkdir /var/www/html/stream/live3
```

<h4> 5 - Verificar e recarregar os arquivos de configurações </h4>

- Verificando os arquivos de configurações
```
sudo nginx -t
```
- Recarregando os arquivos de configurações
```
sudo nginx -s reload
```

<h3> Instalando e executando o ffmpeg </h3>

<h4> 1 - Instalar o pacote ffmpeg</h4>

```
sudo apt -y install ffmpeg
```

<h4> 2 - Executar o processamento e manipulação da mídia </h4>

- Executando os três comandos em três terminais diferentes
```
ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 1 > -c:v libx264 -f flv rtmp://localhost/live1/output

ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 2 > -c:v libx264 -f flv rtmp://localhost/live2/output

ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 3 > -c:v libx264 -f flv rtmp://localhost/live3/output
```
- Ou executando os três comandos no mesmo terminal
    - Criando um arquivo live.sh
    ```
    sudo touch live.sh
    ```
    - Editando o arquivo live.sh
    ```
    sudo vim live.sh
    ```
    - Configurando o arquivo live
    ```
    #!/bin/bash
    
    ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 1 > -c:v libx264 -f flv rtmp://localhost/live1/output &

    ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 2 > -c:v libx264 -f flv rtmp://localhost/live2/output &
    
    ffmpeg -rtsp_transport tcp -i < URL RTSP da camera 3 > -c:v libx264 -f flv rtmp://localhost/live3/output &
    ```
- Ou executando os três comandos como serviços
    - Criando os arquivo lives.service no diretorio /etc/systemd/system/
    ```
    sudo touch /etc/systemd/system/live1.service

    sudo touch /etc/systemd/system/live2.service

    sudo touch /etc/systemd/system/live3.service
    ```
    - live1
        - Editando o arquivo live1.service
        ```
        sudo vim /etc/systemd/system/live1.service
        ```
        - Configurando o arquivo live1.service
        ```
        [Unit]
        Description=Live 1
        Requires=network.target
        After=network.target
        StartLimitIntervalSec=0
        
        [Service]
        Type=simple
        Restart=always
        RestartSec=10
        User=root
        ExecStart=ffmpeg -rtsp_transport tcp -(timeout|stimeout) 10000000 -i < URL RTSP da camera 1 > -c:v libx264 -vf scale=1280:720 -profile:v baseline -level:v 3.0 -f flv rtmp://localhost/live1/output
        
        [Install]
        WantedBy=multi-user.targe
        ```
    - live2
        - Editando o arquivo live2.service
        ```
        sudo vim /etc/systemd/system/live2.service
        ```
        - Configurando o arquivo live2.service
        ```
        [Unit]
        Description=Live 2
        Requires=network.target
        After=network.target
        StartLimitIntervalSec=0
        
        [Service]
        Type=simple
        Restart=always
        RestartSec=10
        User=root
        ExecStart=ffmpeg -rtsp_transport tcp -(timeout|stimeout) 10000000 -i < URL RTSP da camera 2 > -c:v libx264 -vf scale=1280:720 -profile:v baseline -level:v 3.0 -f flv rtmp://localhost/live2/output
        
        [Install]
        WantedBy=multi-user.targe
        ```
    - live3
        - Editando o arquivo live3.service
        ```
        sudo vim /etc/systemd/system/live3.service
        ```
        - Configurando o arquivo live3.service
        ```
        [Unit]
        Description=Live 3
        Requires=network.target
        After=network.target
        StartLimitIntervalSec=0
        
        [Service]
        Type=simple
        Restart=always
        RestartSec=10
        User=root
        ExecStart=ffmpeg -rtsp_transport tcp -(timeout|stimeout) 10000000 -i < URL RTSP da camera 3 > -c:v libx264 -vf scale=1280:720 -profile:v baseline -level:v 3.0 -f flv rtmp://localhost/live3/output
        
        [Install]
        WantedBy=multi-user.targe
        ```
    - Recarregando as configurações
    ```
    sudo systemctl daemon-reload
    ```
    - Ativando os serviços das lives
    ```
    sudo systemctl enable live1.service live2.service live3.service
    ```
    - Inicializando os serviços das lives
    ```
    sudo systemctl start live1.service live2.service live3.service
    ```

<h3> Executando o ffmpeg em GPU NVidia</h3>

<h4>Exemplo:</h4>

```
ffmpeg -y -hwaccel cuda -hwaccel_output_format cuda -rtsp_transport tcp -i < URL RTSP da camera >  -c:a copy -c:v h264_nvenc -vf scale_cuda=1280:720 -r 30 -preset p6 -tune hq -b:v 5M -bufsize 5M -maxrate 10M -qmin 0 -g 250 -bf 3 -b_ref_mode middle -temporal-aq 1 -rc-lookahead 20 -i_qfactor 0.75 -b_qfactor 1.1 -f flv rtmp://localhost/live1/output

```

