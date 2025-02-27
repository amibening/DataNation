# DataNation
DataNation - Turning Dormant Data into Business Assets

Please note - the initial project draft WhitePaper and pitch deck are saved as PDFs in the docs directory. 

# DataNation Smart Contract Design Document

**Overview**

DataNation is a decentralised platform that empowers businesses to monetise their datasets, particularly for training Large Language Models (LLMs), by leveraging Cardano’s blockchain technology. The platform connects dataset owners with AI engineers and data analysts to enhance dataset quality and facilitates a marketplace for selling datasets or fractional ownership stakes. Smart contracts automate token minting, marketplace interactions, income distribution, and dataset buyouts, ensuring transparency, security, and compliance with privacy and regulatory standards.

The smart contracts are designed to:

Mint a reference NFT and 1,000 fractional tokens per dataset using the CIP-68 standard.
Enable trading of fractional tokens on a marketplace.
Manage income from dataset usage and distribute it to fractional owners via a treasury.
Handle outright dataset purchases with proceeds distributed to token holders.
This document outlines the architecture, data structures, and validation logic for these contracts in Aiken.
