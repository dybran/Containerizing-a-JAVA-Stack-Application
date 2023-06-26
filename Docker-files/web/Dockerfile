FROM nginx
LABEL "Project"="Vprofile"
LABEL "Author"="Solomon Onwuasoanya"
LABEL "Description"="Building Vprofileweb image"

RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginvproapp.conf /etc/nginx/conf.d/vproapp.conf