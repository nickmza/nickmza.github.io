---
title: Integrating Azure Storage and SFTP with Apache Camel
tags:
  - Camel
category: Software Engineering
ogimage: csa-1.png
excerpt: >-
  This week I needed to get data from a Power Automate flow to an internal SFTP
  server. We are evaluating a couple of options for this but here's how you
  would do it with Apache Camel...
date: 2023-09-02 09:14:15
---


This week I needed to get data from a Power Automate flow to an internal SFTP server. We are evaluating a couple of options for this but here's how you would do it with Apache Camel.

<iframe frameborder="0" style="width:100%;height:200px;" src="https://viewer.diagrams.net/?highlight=0000ff&nav=1&title=Camel-Azure-SFTP#R3ZZRb5swEMc%2FDY%2BTwFDSPbZpuklbp2xsmvrohgt4MxxyjgD99DPlgCDUKJOmJeoT%2BHdnzv77f8iOv8zqD0YW6QPGoB3hxrXj3zlCeCIQ9tGSpiOL6%2FcdSIyKOWkEkXoGhi7TUsWwmyQSoiZVTOEG8xw2NGHSGKymaVvU06qFTGAGoo3Uc%2FpTxZR29PrKHflHUEnaV%2FZcjmSyT2awS2WM1QHyV46%2FNIjUvWX1EnQrXq9LN%2B%2F%2BleiwMAM5nTJBNV%2B%2Bhb%2B3xeP6x93Xp8Wnh4Xx3vndV%2FZSl7xhXiw1vQIQW0F4iIZSTDCXejXSW4NlHkNbxrWjMeczYmGhZ%2BEvIGr4dGVJaFFKmeZoV7Mt9OreGO2wNBs4sqHeI9IkQEfyxHAC1rqAGZBp7DwDWpLaT9ch2UPJkDfKbF9Y6b9Q3ZupvsYKjEU3VhvrHJidQpUqgqiQL5uvbK9NFZS7onP%2FVtXtSbCkezAE9XFR5yIMzcvO5db1Qh5XB43AKD3ogZ79c9mu3ppZxYlmDc5pVjFT%2Fea5NNahbkRo2t%2BncG81Pl2CZQfvXYplF2%2FNssGJlg3PadlgpvpSZu3F5PwODYILc2g40yq6%2F76%2BBKlC979JZYfjRewldnCd9Vd%2FAA%3D%3D"></iframe>

# Setup
To simplify testing I wanted a local SFTP instance to work with. I used the atmoz/sftp image to bring up one using Docker.

```
docker run -p 22:22 -d atmoz/sftp foo:pass:::upload
```

Once this is running you can log into the local sftp as follows:
```
sftp -P 22 foo@127.0.0.1  
```

# Azure Storage Blob    

Connecting to Azure Storage is straight-forward. Add a reference to the [Azure Storage Blob Component](https://camel.apache.org/components/4.0.x/azure-storage-blob-component.html#_message_headers).

Then in your route configure the Blob Client:

```
private void configureBlobClient() {
    StorageSharedKeyCredential credential = new StorageSharedKeyCredential(config.getStorageAccountName(), config.getAccessKey());
    String uri = String.format("https://%s.blob.core.windows.net", config.getStorageAccountName());

    BlobServiceClient client = new BlobServiceClientBuilder()
            .endpoint(uri)
            .credential(credential)
            .buildClient();

    context.getRegistry().bind("client", client);
}
```
Once this is done you can receive Blobs as follows:

```
from("azure-storage-blob://your_storage_account/your_folder?serviceClient=#client")
    .log(LoggingLevel.INFO, "File Received: $simple{in.header.CamelAzureStorageBlobBlobName}")

```
The code above receives all Blobs added to the 'your_folder' container in 'your_storage_account'. You can specify the filename or a Regex pattern to further refine which blobs are received. See the [Component Details](https://camel.apache.org/components/4.0.x/azure-storage-blob-component.html#_component_options) section.

# SFTP
To enable SFTP add a reference to the [SFTP](https://camel.apache.org/components/4.0.x/sftp-component.html#_more_information) component.

You can then add configuration at Component or Endpoint level. Here's our endpoint:
```
.to("sftp:1.1.1.1/files/sharepoint?username=myuser&password=mypassword")
```
This works fine but we'd probably want to tighten up on how we handle the password and host verification for production. 

An additional step is to configure the Filename we want to use for the file. By default Camel will auto-generate one. We wanted to use the same name as the blob. This can be done by setting the appropriate header values:
```
.process(exchange -> {
    String filename = exchange.getIn().getHeader(BlobConstants.BLOB_NAME, String.class);
    exchange.getIn().setHeader(FtpConstants.FILE_NAME, filename);
})
```
# Cleaning up the Blob
Once we have uploaded the file to SFTP we need to delete the original blob - otherwise Camel will continue to receive it. 
```
.to(config.getSourceUri() + "&operation=deleteBlob")
```
The Blob component figures out which Blob to delete based on the BlobConstants.BLOB_NAME header. You can also specify which Blob to delete in the route itself.

# Bringing it all together.
Here's the final route:
```
    from(config.getSourceUri()) 
        .routeId(FUSION_GL_UPDATE) 
        .log(LoggingLevel.INFO, "File Received: $simple{in.header.CamelAzureStorageBlobBlobName}") 
        .log(LoggingLevel.DEBUG, "Contents: ${body}") 
        .process(exchange -> { 
            String filename = exchange.getIn().getHeader(BlobConstants.BLOB_NAME, String.class); 
            exchange.getIn().setHeader(FtpConstants.FILE_NAME, filename); 
        }) 
        .to(config.getDestinationUri()) 
        .to(config.getSourceUri() + "&blobName=blob&operation=deleteBlob") 
        .log(LoggingLevel.INFO, "File delivered."); 
```

# Conclusion
This is a great example of how Camel simplifies common integration patterns and flows. There's still some work to do in terms of hardening and we may want to add some logic to prevent duplicate uploads but the core of this was up and running in about an hour or so.