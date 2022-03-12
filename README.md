# Cloud Security


We have essentially three choices of provider. Regardless of which cloud we use, we strive to be efficient, to 
yield the maximum compute for each dollar spent. Efficiency is a combination of cost management (best practices
using cloud resources) and security (granting access to resources precisely and only to those who need it). 
This repository is concerned with the latter.

## Why use a particular cloud 

The choice of cloud is often connected to the task of building infrastructure for a specific project. 
This choice is often a matter of convenience, for example based on particular features and services 
offered by that cloud; or based on our prior experience and expertise. 


We emphasize that equivalent levels of security and reliability are available
from all public cloud platforms. For this reason we do not
endorse a particular platform. Ideally we will build this material out as three parallel constructions; 
hence we begin with three corresponding folders. 


## What is *unwanted use*?


***Unwanted use*** takes on several forms. First if an external agency gets working access keys to a cloud
account they can quickly start very expensive resources. The typical example is using VMs to mine bitcoin. 
Second, if an external agency gains access to data that includes identities of real human beings then we
have failed to comply with federal regulations that strictly forbid this. Third if an external agency
gains access to data we consider valuable, they can encrypt or otherwise embargo that data and demand 
some form of ransom. There are other forms of unwanted use of cloud resources as well. The corollary rule
to be aware of, towards preventing unwanted use, is the following: 

> ***Never take action on the cloud unless and until you are certain that action does not create significant risk.***


What this means in practice is: Training. You and your entire research team must be able to commit to 
learning the ropes of secure practice on the cloud. 


## Where next? 

Each sub-folder contains material for security practices on the respective clouds; or placeholders for such material.
