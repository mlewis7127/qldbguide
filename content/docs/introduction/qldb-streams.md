---
title: "QLDB Streams"
date: 2020-07-10T18:41:23Z
lastmod: 2020-07-11
weight: 35
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

{{< codeblock "language-json" >}}
{
    "DigestTipAddress": {
        "IonText": "{strandId:\"GadsaE07Jm09FAL3odPrDx\",sequenceNo:277}"
    }, 
    "Digest": "4zXgZ/7+e0jKpxhnNcF7ydYGMoEsRbOH84QbVz40w2w="
}
{{< /codeblock  >}}

The response includes the SHA-256 hash value representing the digest, and a block address of the latest block location covered by the digest. This consists of the strand ID and the sequence number for the location in the chain.

**Step 2 - Get Block Address and ID**

The second step is to get hold of the block address and the document ID for the specific document revision to be verified. An example PartiQL statement is shown below:

{{< codeblock "language-shell" >}}
SELECT blockAddress, metadata.id FROM _ql_committed_{TABLE_NAME} WHERE data.{ID} = ?
{{< /codeblock  >}}


**Step 3 - Get Revision**

The third step is to retrieve a Proof object for the specific document revision. This can be carried out using the `getRevision` method on the QLDB service in the AWS SDK. This is also available on the CLI. This method passes in the following parameters in the request body to the specified ledger:

{{< codeblock "language-json" >}}
{
   "BlockAddress": { 
      "IonText": "string"
   },
   "DigestTipAddress": { 
      "IonText": "string"
   },
   "DocumentId": "string"
}
{{< /codeblock  >}}

The BlockAddress and DocumentID are the values retrieved in Step 2, and the DigestTipAddress is retrieved in Step 1.

If successful, the service returns a Proof object in Amazon ION format, and the document revision data object in Amazon ION format.

The Proof object contains the list of hashes required to recalculate the specified digest using a Merkle tree, starting with the specified document revision. An example of a Proof object is shown below:

{{< codeblock "language-shell" >}}
[
    {{3LTcVdBpFhb/B2WVheUF+hP4RrXEMs+iGU2/UXpcx7U=}},
    {{J4aLw+08OWOFcmnGHdzduhaX+f/IDuKPjkry/muZJ1w=}},
    {{D35JeemUXb8hE9IQQUjKKDuIOCzitdv2xhOMeNzX0rc=}},
    {{D57JeemUXb8hE9IQQUjKKEuIOCzitdv2xhOMeNzX0rc=}},
    {{Yuv58uEca488TqS5YBaZB/bcNfEH3+JxU79LRMM0YzY=}},
    {{VycOpIKduiX9H3JLQeGYo0ABAdnocS57t5y6xT63eK0=}},
    {{Fd4k5s/GRmJKz8TdQD+ztTlt+RbuR5F/7qDj2gR920w=}},
    {{C7VoQEuqoEEmiog6dXVKWoh616Wqql+XxU9q+5SJpXA=}},
    {{W53IoMzXf/bpxEZ7LJKUMKrtJnPCWsIf7h5hfpECfEE=}},
    {{8hlC9sstorH7Nno1Xs9loba9mCrTvNCNTOEJS5vKaN0=}}
]
{{< /codeblock  >}}

An example of the document revision object is shown below:

{{< codeblock "language-terminal" >}}
{
    blockAddress:{
      strandId:"GadsaE07Jm09FAL3odPrDx\",
      sequenceNo:151
    },
    hash:{{mq1El5k5Rxljgea71ZSYSZwbwBvm81vbJs6zsiRDqgc=}},
    data:{
      ...
    },
    metadata:{
      id:"AO55UerdfaW7KzAQiMxgQl",
      version:1,
      txTime:2020-03-30T14:32:41.020Z,
      txId:"ElrCO57nOi2BKvhrDUPyIS"
    }
}
{{< /codeblock  >}}


**Step 4 - Recalculate Digest**

The next step is to build up a candidate digest to represent the entire ledger by using the Proof hashes. 

This starts off by taking the hash from the document revision, and pairing it with the first hash in the Proof hashes list. These two hash values are sorted, concatenated, and then a new hash value generated from the concatenated values. This essentially uses a reduce function, by carrying out this action on each element in the hashes list, until only a single hash value remains. This is the recalculated digest. 


**Step 5 - Compare Digest Values**

The final step is to compared the recalculated digest from the previous step, with the digest value returned in the first step. If both values are the same, then we have successfully verified the record. 


## Streaming

At Reinvent 2019, AWS announcing the private preview of QLDB Streaming

![Streaming Preview](/images/QLDBStreaming.svg)


