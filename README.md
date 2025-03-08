# governance-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract Governance {
    struct Proposal {
        string description;
        uint256 votesFor;
        uint256 votesAgainst;
        bool executed;
        mapping(address => bool) voted;
    }
    
    address public admin;
    uint256 public proposalCount;
    mapping(uint256 => Proposal) public proposals;
    mapping(address => uint256) public votingPower;
    
    event ProposalCreated(uint256 proposalId, string description);
    event Voted(uint256 proposalId, address voter, bool support);
    event ProposalExecuted(uint256 proposalId);
    
    modifier onlyAdmin() {
        require(msg.sender == admin, "Not an admin");
        _;
    }
    
    constructor() {
        admin = msg.sender;
    }
    
    function setVotingPower(address voter, uint256 power) external onlyAdmin {
        votingPower[voter] = power;
    }
    
    function createProposal(string calldata _description) external onlyAdmin {
        proposalCount++;
        proposals[proposalCount] = Proposal(_description, 0, 0, false);
        emit ProposalCreated(proposalCount, _description);
    }
    
    function vote(uint256 _proposalId, bool support) external {
        Proposal storage proposal = proposals[_proposalId];
        require(!proposal.executed, "Proposal already executed");
        require(!proposal.voted[msg.sender], "Already voted");
        require(votingPower[msg.sender] > 0, "No voting power");
        
        proposal.voted[msg.sender] = true;
        if (support) {
            proposal.votesFor += votingPower[msg.sender];
        } else {
            proposal.votesAgainst += votingPower[msg.sender];
        }
        
        emit Voted(_proposalId, msg.sender, support);
    }
    
    function executeProposal(uint256 _proposalId) external onlyAdmin {
        Proposal storage proposal = proposals[_proposalId];
        require(!proposal.executed, "Already executed");
        require(proposal.votesFor > proposal.votesAgainst, "Not enough support");
        
        proposal.executed = true;
        emit ProposalExecuted(_proposalId);
    }
}
890
