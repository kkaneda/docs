Open WebUI
==========

`Open WebUI <https://docs.openwebui.com/>`_ provides a web UI that works with OpenAI-compatible APIs. You can run Openn WebUI locally or
run in a Kubernetes cluster.

Here is an instruction for running Open WebUI in a Kubernetes cluster.

.. code-block:: console

   OPENAI_API_KEY=<LLMariner API key>
   OPEN_API_BASE_URL=<LLMariner API endpoint>

   kubectl create namespace open-webui
   kubectl create secret generic -n open-webui llm-operator-api-key --from-literal=key=${OPENAI_API_KEY}

   kubectl apply -f - <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: open-webui
     namespace: open-webui
   spec:
     selector:
       matchLabels:
         name: open-webui
     template:
       metadata:
         labels:
           name: open-webui
       spec:
         containers:
         - name: open-webui
           image: ghcr.io/open-webui/open-webui:main
           ports:
           - name: http
             containerPort: 8080
             protocol: TCP
           env:
           - name: OPENAI_API_BASE_URLS
             value: ${OPEN_API_BASE_URL}
           - name: WEBUI_AUTH
             value: "false"
           - name: OPENAI_API_KEYS
             valueFrom:
               secretKeyRef:
                 name: llm-operator-api-key
                 key: key
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: open-webui
     namespace: open-webui
   spec:
     type: ClusterIP
     selector:
       name: open-webui
     ports:
     - port: 8080
       name: http
       targetPort: http
       protocol: TCP
   EOF

You can then access Open WebUI with port forwarding:

.. code-block:: console

   kubectl port-forward -n open-webui service/open-webui 8080
