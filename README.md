using System;
using System.Collections.Generic;
using System.Security.Cryptography;
using System.Text;
using System.Linq;

// Represents a single transaction
public class Transaction
{
    public string Sender { get; set; }
    public string Recipient { get; set; }
    public double Amount { get; set; }

    public override string ToString()
    {
        return $"Sender: {Sender}, Recipient: {Recipient}, Amount: {Amount}";
    }
}

// Represents a block in the blockchain
public class Block
{
    public int Index { get; set; }
    public DateTime Timestamp { get; set; }
    public string PreviousHash { get; set; }
    public string Hash { get; set; }
    public int Nonce { get; set; }
    public List<Transaction> Transactions { get; set; } = new List<Transaction>();

    public Block(int index, DateTime timestamp, string previousHash)
    {
        Index = index;
        Timestamp = timestamp;
        PreviousHash = previousHash;
        Nonce = 0; // Initial nonce
    }

    public void CalculateHash()
    {
       string data = $"{Index}{Timestamp}{PreviousHash}{Nonce}{string.Join("", Transactions.Select(t => t.ToString()))}";

      using (SHA256 sha256 = SHA256.Create()) {
          byte[] hashBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(data));
           Hash =  BitConverter.ToString(hashBytes).Replace("-","").ToLower();
      }
    }
    public override string ToString()
    {
        return $"Index: {Index}, Timestamp: {Timestamp}, PreviousHash: {PreviousHash}, Hash: {Hash}, Nonce:{Nonce} Transactions: {string.Join(" | ", Transactions.Select(t => t.ToString()))}";
    }
}

// Represents the blockchain
public class Blockchain
{
    public List<Block> Chain { get; set; } = new List<Block>();
    public int Difficulty { get; set; } = 2;

    public Blockchain()
    {
        // Create the genesis block
        CreateGenesisBlock();
    }

     private void CreateGenesisBlock()
    {
        Block genesisBlock = new Block(0, DateTime.Now, "0");
        genesisBlock.CalculateHash();
        Chain.Add(genesisBlock);
    }

    public Block GetLatestBlock() {
         return Chain.Last();
    }

      public void AddTransaction(string sender, string recipient, double amount) {

          if(String.IsNullOrEmpty(sender) || String.IsNullOrEmpty(recipient) || amount <= 0) {
              Console.WriteLine("Invalid Transaction data");
              return;
          }

         Transaction transaction = new Transaction { Sender = sender, Recipient = recipient, Amount = amount };
         GetLatestBlock().Transactions.Add(transaction);
         Console.WriteLine($"Transaction added: {transaction}");
      }

    // Adds a new block to the chain using proof-of-work
     public void AddBlock( )
    {
         Block previousBlock = GetLatestBlock();
          Block block = new Block(previousBlock.Index + 1, DateTime.Now, previousBlock.Hash);

          //add all pending transactions to the block
           block.Transactions.AddRange(previousBlock.Transactions);
           previousBlock.Transactions.Clear();

           //proof of work
           MineBlock(block);

        Chain.Add(block);

        Console.WriteLine($"Block Added: {block}");
    }


    public void MineBlock(Block block)
    {
        while (!block.Hash.StartsWith(new string('0', Difficulty)))
        {
           block.Nonce++;
           block.CalculateHash();
        }
        Console.WriteLine($"Block mined successfully with nonce: {block.Nonce}");
    }

    // Check if chain is valid
      public bool IsChainValid() {
         for(int i = 1; i < Chain.Count; i++)
         {
              Block currentBlock = Chain[i];
              Block previousBlock = Chain[i - 1];

             string currentHash = currentBlock.Hash;
              currentBlock.CalculateHash();
              if(currentBlock.Hash != currentHash ) {
                 Console.WriteLine($"Invalid Block at Index: {currentBlock.Index}. Hashes do not match.");
                 return false;
              }


              if(currentBlock.PreviousHash != previousBlock.Hash) {
                   Console.WriteLine($"Invalid Block at Index: {currentBlock.Index}. Previous block hashes do not match.");
                   return false;
              }
         }
         return true;
      }

    public void DisplayChain()
    {
        Console.WriteLine("\nBlockchain:");
         foreach(var block in Chain)
        {
            Console.WriteLine(block);
             Console.WriteLine("--------------------");
        }
    }
}


public class BlockchainSimulator
{
    public static void Main(string[] args)
    {
        Blockchain blockchain = new Blockchain();

        while (true) {
            Console.WriteLine("\nOptions:");
            Console.WriteLine("1. Add Transaction");
            Console.WriteLine("2. Mine Block");
            Console.WriteLine("3. Display Blockchain");
            Console.WriteLine("4. Validate Chain");
            Console.WriteLine("5. Exit");

            Console.WriteLine("Enter your choice: ");
              if (!int.TryParse(Console.ReadLine(), out int choice)) {
                    Console.WriteLine("Invalid input. Please enter a number.");
                    continue;
            }

            switch (choice) {
                 case 1:
                     Console.WriteLine("Enter sender name:");
                     string sender = Console.ReadLine();
                    Console.WriteLine("Enter recipient name:");
                     string recipient = Console.ReadLine();
                     Console.WriteLine("Enter Amount:");
                     if (!double.TryParse(Console.ReadLine(), out double amount)){
                         Console.WriteLine("Invalid amount entered.");
                        break;
                     }
                     blockchain.AddTransaction(sender, recipient, amount);
                     break;
                case 2:
                   blockchain.AddBlock();
                   break;
                 case 3:
                    blockchain.DisplayChain();
                    break;
                case 4:
                  bool isValid =   blockchain.IsChainValid();
                  if(isValid) {
                      Console.WriteLine("Chain is Valid.");
                   } else {
                      Console.WriteLine("Chain is Invalid.");
                   }
                   break;
                case 5:
                    Console.WriteLine("Exiting...");
                     return;
                default:
                    Console.WriteLine("Invalid choice.");
                     break;

            }

        }


    }
}
