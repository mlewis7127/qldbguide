---
title: "Advanced Topics"
date: 2020-02-21T18:41:23Z
lastmod: 2020-02-21
weight: 30
draft: false
---

# Advanced Topics

## Crytographic Verification

One of the most powerful features of QLDB is the ability to crytographically verify that a document revision (e.g. the state of a record at a particular point in time). This proves that the record exists at the same location in the journal and has not been altered in any way, since it was first written.

**Step 1 - Get Digest**
The first step to carry out verification is to retrieve the digest. This can be carried out using the `getDigest` method on the QLDB service in the AWS SDK. This is also available on the CLI:

{{< codeblock "language-shell" >}}
aws qldb get-digest --name qldb-guide
{{< /codeblock  >}}

This method returns the digest of the ledger at the latest committed block in the journal. An example response is shown below:

{{< codeblock "language-shell" >}}
{
    "DigestTipAddress": {
        "IonText": "{strandId:\"GadsaE07Jm09FAL3odPrDx\",sequenceNo:277}"
    }, 
    "Digest": "4zXgZ/7+e0jKpxhnNcF7ydYGMoEsRbOH84QbVz40w2w="
}
{{< /codeblock  >}}

The response includes the SHA-256 hash value representing the digest, and a block address of the latest block location covered by the digest. This consists of the strand ID and the sequence number for the location in the chain.

**Step 2 - Get Block Address and ID**


**Step 3 - Get Revision**


**Step 4 - Recalculate Digest**


**Step 5 - Compare Digest Values**



## Streaming

At Reinvent 2019, AWS announcing the private preview of QLDB Streaming

![Streaming Preview](/images/QLDBStreaming.svg)


