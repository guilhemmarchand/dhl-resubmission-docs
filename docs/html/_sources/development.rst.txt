Development and build process
#############################

**The Application is built on top of the Splunk Addon factory generator, which allows to build and maintain consistent and modular Splunk applications using the most up to date content:**

- https://github.com/splunk/addonfactory-ucc-generator

- https://splunk.github.io/addonfactory-ucc-generator/

For development purposes, different dependencies should be installed on the development platform, as follows.

Dependencies for developers
===========================

- Operating system: Linux or MacOS (seriously, what else?)

- Access to the machine in terminal (SSH if remote)

- Python 3.7 or later

- Splunk ucc-gen

ucc-gen and updating applications
=================================

**The development of the Splunk applications relies on the Splunk UCC generator:**

- https://github.com/splunk/addonfactory-ucc-generator

- https://splunk.github.io/addonfactory-ucc-generator/

**When performing changes in the application, you will:**

- update the ``version.json`` file to increment the version release number

- perform your changes effectively, by managing files in the ``<application_name>/package`` directory (such as modifying a view, etc)

- run the ucc-gen command via a shell wrapper:

::

    cd <application_name>/build
    python3 build.py --keep

- This produces automatically a new packaged application, located in the ``<application_name>/output/`` directory

- The tgz package is ignored in GitHub on purpose, you will take this new tarball archive, publish it as a new release in the repository releases, and finally deploy to your Splunk environment

*example:*

::

    <application_name>/
    <application_name>_<version>_<build>.tgz
    release-sha256.txt

- the content of the directory is the uncompress content from the generated package (and is ignored in Git on purpose too)

- the tgz file is the package to be released in GitHub, and deployed to Splunk

MQ developments with Docker
===========================

**For easy and modern developping purposes, one can easily bootstrap an environment including Splunk and MQ-Series using docker, for your convenience, a docker-compose.yml example is shared as follows:**

*docker-compose.yml, adapt to your needs, create or remove local storage references as needed:*

*the following docker configuration allows bootstrapping a ready to be used MQ manager:*

::

    version: "3.8"

    # services
    services:

    # IBM MQ
    mq1:
        image: ibmcom/mq:latest
        hostname: mq1
        ports:
        - "1414:1414"
        - "9443:9443"
        environment:
        LICENSE: "accept"
        MQ_QMGR_NAME: "QM1"
        MQ_ADMIN_PASSWORD: "gv51JMYghk1D5%M9"
        MQSNOAUT: 'YES'
        # If the following is set, authentication will be required
        #MQ_APP_PASSWORD: "passw0rd"
        volumes:
        - "qm1data:/mnt/mqm:rw"

    volumes:
    qm1data: {}

SOAP API developments
=====================

**The following Docker configuration were used to bootstap a SOAP API, with an nginx reverse proxy for the SSL layer:**

::

    version: "3.8"
    services:

    soap:
        image: python:latest
        restart: unless-stopped
        hostname: soap
        volumes:
        - ./soap:/soap
        command: "/soap/run.sh"
        #command: "tail -f /dev/null"

    # Nginx acting as the SSL proxy
    nginx:
        image: nginx:latest
        restart: unless-stopped
        hostname: nginx
        volumes:
        - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
        - ./nginx/ssl.cert:/etc/nginx/cert/ssl.cert
        - ./nginx/ssl.key:/etc/nginx/cert/ssl.key
        - ./nginx/.htpasswd:/etc/nginx/.htpasswd
        ports:
        - 8081:8081
        - 443:443

Which requires the following content in the directory "soap":

soap/
     run.sh
     run.py
nginx/
      default.conf
      ssl.cert
      ssl.key

*File run.sh:*

::

    #!/bin/bash

    python3 -m pip install flask
    python3 /soap/run.py    

*File soap/run.py:*

::

    from flask import Flask, request, make_response

    app = Flask(__name__)

    @app.route('/soap-endpoint', methods=['POST'])
    def soap_endpoint():
        # Get the XML data sent to the server
        xml_data = request.data

        # Here you can parse the XML and construct a response or just print it
        print(xml_data)

        # For the purpose of this demonstration, let's return a generic SOAP response.
        response = make_response("""<?xml version="1.0" encoding="UTF-8"?>
    <ns63:ShpPcdDetailsMsg xmlns:ns63="http://www.dhl.com/Express/CM/CM_ShpPcsDetailsMsg/v2" xmlns:OAGIS="http://www.openapplications.org/oagis/9" xmlns:ns5="http://www.dhl.com/Express/CM/CM_GenericRequest/v2">
    <Hdr Id="593ae885-907e-4cb0-aa50-eb16540e4f10" Ver="2.001" Dtm="2023-06-08T07:08:15">
        <GI>
        <TID Src="ESB" TID="e7c35e05-4cb6-496b-aa72-b2eaa820fc97"/>
        </GI>
        <Sndr AppCd="ESB" AppVer="3.0" AppNm="ESB"/>
        <Rcp AppCd="ESB" AppVer="1.0" AppNm="ESB">
        <GId Id="46069296" IdTp="RQMSGID"/>
        </Rcp>
        <Orgntr AppCd="EAPI" AppVer="1.0" AppNm="MYDHL API"/>
    </Hdr>
    <Bd/>
    <RespStatus>
        <GI CorrId="1931182186"/>
        <Sev>10</Sev>
        <MsgCIgd>200000</MsgCIgd>
        <MsgClg>IPESB</MsgClg>
        <Dsc>ShpKey is Valid.</Dsc>
        <DtlDsc>shipment key is used: 1931182186</DtlDsc>
    </RespStatus>
    <RespStatus>
        <Sev>10</Sev>
        <MsgCIgd>200200</MsgCIgd>
        <MsgClg>1931182186</MsgClg>
        <Dsc>Successfully put to DCVS3_ASIA MQ Queue. Response code: 0</Dsc>
        <DtlDsc>IPESB</DtlDsc>
    </RespStatus>
    <RespStatus>
        <GI CorrId="46069296"/>
        <Sev>10</Sev>
        <MsgCIgd>100000</MsgCIgd>
        <MsgClg>IPESB</MsgClg>
        <Dsc>Service operation execution finished successfully</Dsc>
        <DtlDsc>Service operation execution finished successfully</DtlDsc>
        <RpBy CmptNm="ShipmentManagement.ShipmentManifestProvider_v4"/>
    </RespStatus>
    </ns63:ShpPcdDetailsMsg>""",200)
        response.headers['Content-Type'] = 'text/xml'
        return response

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)    

*File nginx/default.conf:*

::

    server {
            listen 8081;
            listen [::]:8081;

            access_log /var/log/nginx/reverse-access.log;
            error_log /var/log/nginx/reverse-error.log;


            ssl on;
            ssl_certificate /etc/nginx/cert/ssl.cert;
            ssl_certificate_key /etc/nginx/cert/ssl.key;

        client_max_body_size 64M;

            location / {
                        auth_basic "Restricted Content";
                auth_basic_user_file /etc/nginx/.htpasswd;
                proxy_pass http://soap:5000;
    }
    }

Finally, the ssl.cert and ssl.key are simply self-signed SSL certificate and no passphase private key created for the purpose of the SSL layer.
