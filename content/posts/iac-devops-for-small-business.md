+++
title = 'IAC (+ DevOps) for small business'
date = 2024-04-18T09:37:01+02:00
draft = false
+++

This article is an analysis of how to approach **Infrastructure as Code** (IAC) for a small business. I'm a co-owner of a small startup company responsible for the application for tourists and guides called [Locura](https://play.google.com/store/apps/details?id=cz.projectport.locura&hl=en&gl=US) and I've been trying to put all my experience to use as a developer, devops and an architect from other companies I've worked for.

## Starting in startup
At the beggining of the Locura project I imagined that I'll never put up with the physical layer and everything would run on cloud, in my case Azure. Not only that --- my idea was that everything would be PaaS. I do not want to manage VMs either. Just give me a platform to deploy on and let me work. And everything was to be smooth, at least that's what I imagined. No physical infrastructure, no VMs, only pure PaaS with Azure WebApps.

![Everything on Azure](/posts/images/iacdevops/azure1.png)

And of course very soon I learned this is not really possible unless I want to pay a lot of money. Because the awesome thing with clouds like Azure is that you can provision everything very quickly. This leads you to thinking that **it's easy** is more important than **it's expensive**.

![Expensive choices on Azure](/posts/images/iacdevops/azurepricey.png)

I like cloud. I like how everything can be provisioned quickly and how you can have a lot of infrastructure almost immediately ready. With tools like terraform you can update it or remove it as quickly as well.

Can you do something about it? For example, instead of provisioning multiple WebApp Plans (each one costs money) you provision only a single one and all environments are deployed into the same plan?

![Azure plans](/posts/images/iacdevops/azureplan.png)

The problem with PaaS is that you don't get to utilize all the CPU/RAM of the machine on which the platform is running on. Because the *platform*, in **Platform** As A Service, is not a single thing. It is a whole set of applications that the cloud provider is running in the background so you can have your PaaS with all the PaaS advantages (easy monitoring, easy configuration, easy everything else...).

So for B1 basic Linux plan with 1 core and 1,75 GB RAM for which you pay cca $14/month you need to understand that you don't have a platform with 1,75 GB RAM freely available. For this little RAM almost 50-60% of that is already occupied by the platform itself so you have only half of it which is around 700 MB.

This is false advertisment. For the PaaS platforms the cloud providers should publish how much RAM in average their platform utilises on a given plan. So instead of seeing 1,75 GB RAM for B1 Linux plan, I would like to see something like *1,75 GB RAM (50% available)*. Not only that, a graph should be present showing how much RAM the platform is consuming for each additional app.

Long story short, I couldn't stuff all my environments into a single Azure WebApp plan simply because there was not enough RAM for all environments. This was before `<PublishAot>` option became available in .NET and that's why `<PublishAot>` is so important because the reduction of RAM consumption is [big](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot?view=aspnetcore-8.0). At the lower price ranges in Azure the ability of your app to be compiled AOT is suddenly not only a cool feature to have but a really important property.

Also you should not really put other environments on the same machine as production, that leads to other problems, for obvious reasons, like a load test environment.

## Giving up on Cloud for everything

I realized that my setup in Azure still makes sense but only for the production environment. Everything else would run on a "company infrastructure" which in the case of my startup means "it runs on old PCs I found at my home".

![Azure and other environments](/posts/images/iacdevops/azure_local.png)

So now I have a **hybrid environment**. Something runs on a cloud, something on a local infrastructure behind a VPN.

It's not all so bad though. A lot of cool software that is useful for startups is completely free but you have to run it somewhere locally ...or you run it for free on some proprietary "cloud" with limitations like "only 1 user".

## Ideal IAC

Let's talk about what is an ideal IAC. It's incredibly easy to IAC ...if everything you are operating can be placed in a single cloud such as Azure. Your applications and all their environments. And everything else.

Having everything in the cloud requires 💰money💰 which is bad for a startup but not that bad for a company of certain size. The ease with which you can just code all infrastructure in terraform is just incredible and having your infrastructure fully coded --- and by coded I also mean documented --- in my opinion at certain point offsets the costs of managing any other infrastructure.

## The point of IAC

Any developer worth his money will tell you: the best code is self documenting. Variable names, classes, all identifiers have names that **make sense**. You understand the semantics immediately and you don't need to spend time analysing what was is the purpose of any given code.

If you want to have Infrastructure As Code then the best IAC code is the one that is **self documenting**. Terraform does a pretty good job but it is still quite far from ideal.

## How do you IAC something you cannot IAC?

For example your application depends on some external service running as SaaS. They may have some API and even their own terraform provider but you have to create the account manually because most of such services require an email validation, sometimes even a phone validation. Sometimes they do not have a terraform provider ...or an API to interact with at all!

Sometimes things just require that you do some other things **manually**. And terraform [does not agree with me](https://github.com/hashicorp/terraform/issues/32062#issuecomment-1439133883).

## How do you IAC for IAC?

If you run stuff on Azure you need to have a Microsoft account and you need to use some of many ways of telling terraform how to authenticate with Azure.

But that leads to another problem. Where do you place your terraform authentication to Azure? Terraform provider offers you multiple approaches, from using your local credentials created with `az login` or by specifying a custom `client_id` and `client_secret`. 

But both approaches require manual steps. How do you include that in your IAC? It cannot be part of terraform because it's a step that your terraform project depends on. And this dependency is still part of your architecture and in my opinion should be also part of your IAC.

## Human opinionated external services

[Locura](https://play.google.com/store/apps/details?id=cz.projectport.locura&hl=en&gl=US) is a mobile application deployed on Google Play for Android and on App Store for iOS. And let me tell you that there are lot of hoops you have to jump through **manually** when you are a new company deploying a new application to either of these two application stores. You have to interact with the store employees, Apple even called me at one point, possibly to verify that I'm a real person. And you also have to pay some money, $25 to Google once, $100 to Apple every year.

App stores such as Google Play & App Store on one hand have pretty strict security requirements and on the other hand they don't really want people to be scared of publishing their apps so they have to look at you, your company and your application manually...and make a opinionated decision sometimes even repeatedly in multiple steps.

## IAC Workflows

In some setups it's not possible to have everything written in terraform and call it a day. It's even more complicated than that. Sometimes you need to invoke some scripts AND do manual stuff in certain order. Sometimes there is even conditional logic. And then you can call the terraform project, which, obviously, is now acting as IAC only for a part of your architecture.

These multi-step processes are workflows with manual steps. Is there a good IAC tool that supports workflows? Haha, no!

## IAC configuration & secrets

You have to configure stuff and you have to include secrets in your stuff.

At the top of IAC there is some "root configuration". For example, if you want to provision Azure, you have to provision the information about how to get into Azure. If you want to provision a physical machine running Linux, you have to provision the information about how to get there (e.g. SSH credentials). 

Most of the time, this "root configuration" must be provisioned manually, you cannot automate it. You have to manually create your MS account, set up your Linux user and its password.

### Secrets rotation

Good secrets should rotate with some periodicity that makes sense from security standpoint but is also practical. I guess that in small architectures you can have static secrets as well and there is no problem with that.

But HTTPS certificates force you to rotate some secrets, namely your Let's Encrypt certificates validated with a browser-trusted authorities for your websites. Luckily there are tools such as `acme.sh` or *Nginx Proxy Manager* that help you with it...but again, this is a tool that needs to be downloaded, installed and used which means it's part of your architecture which means it should be mentioned in your IAC somehow.

## Purpose of IAC

The purpose of IAC for me is more than just having a code in one place.

IAC is:
- You have all code in one place
- The code is self documenting
- The code manipulates your architecture **completely** where possible, as terraform does
- The code **documents** your architecture where automated manipulation is not possible
- This means: your code **documents** your architecture **completely**

And of course you should be able to scale this and put your IAC as part of another IAC or to split IAC into many multiple IAC.   
