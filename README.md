# crowdfunding-backened
const db = require("../models");

/**
 * Seeds default campaigns into the database (development only).
 */
const seedCampaigns = async () => {
  try {
    const existingCount = await db.Campaign.countDocuments();
    if (existingCount === 0) {
      const defaultItems = [
        {
          title: "test1",
          subTitle: "subTitle1",
          description: "test1 description here...",
          imageUrl:
            "https://image.shutterstock.com/image-photo/bright-spring-view-cameo-island-260nw-1048185397.jpg",
          required: 500,
          start: new Date("2020-12-22T11:18:54.919Z"),
        },
        {
          title: "test2",
          subTitle: "subTitle2",
          description: "test2 description here...",
          imageUrl:
            "https://image.shutterstock.com/image-photo/bright-spring-view-cameo-island-260nw-1048185397.jpg",
          required: 100,
          start: new Date("2020-12-20T11:18:54.919Z"),
        },
        {
          title: "test3",
          subTitle: "subTitle3",
          description: "test3 description here...",
          imageUrl:
            "https://image.shutterstock.com/image-photo/bright-spring-view-cameo-island-260nw-1048185397.jpg",
          required: 5000,
          start: new Date("2020-12-19T11:18:54.919Z"),
        },
        {
          title: "test4",
          subTitle: "subTitle4",
          description: "test4 description here...",
          imageUrl:
            "https://image.shutterstock.com/image-photo/bright-spring-view-cameo-island-260nw-1048185397.jpg",
          required: 50000,
          start: new Date("2020-12-22T11:19:54.919Z"),
        },
      ];

      await db.Campaign.insertMany(defaultItems);
      console.log("✅ Default campaigns added successfully.");
    }
  } catch (error) {
    console.error("❌ Error seeding campaigns:", error);
  }
};

// Run in development mode only
if (process.env.NODE_ENV === "development") {
  seedCampaigns();
}

/**
 * Hides sensitive transaction IDs in donor records.
 * @param {Array} donors - Array of donor objects
 */
const hideTransactionID = (donors) => {
  if (!Array.isArray(donors)) return;

  donors.forEach((donor) => {
    if (donor.transactionID) {
      donor.transactionID =
        donor.transactionID.slice(0, 4) + "XXXX" + donor.transactionID.slice(-3);
    }
  });
};

/**
 * Fetch a campaign by its ID.
 */
const show = async (req, res) => {
  try {
    const { id } = req.params;

    // Validate ObjectId format
    if (!id.match(/^[0-9a-fA-F]{24}$/)) {
      return res.status(400).json({ message: "Invalid Campaign ID format" });
    }

    const campaign = await db.Campaign.findById(id);
    if (!campaign) {
      return res.status(404).json({ message: "Campaign not found" });
    }

    hideTransactionID(campaign.donors);
    return res.status(200).json(campaign);
  } catch (error) {
    console.error("❌ Error fetching campaign:", error);
    return res.status(500).json({ message: "Internal Server Error" });
  }
};

/**
 * Fetch all campaigns sorted by start date (descending).
 */
const showAll = async (req, res) => {
  try {
    const campaigns = await db.Campaign.find({}).sort({ start: -1 });

    campaigns.forEach((campaign) => {
      hideTransactionID(campaign.donors);
    });

    return res.status(200).json(campaigns);
  } catch (error) {
    console.error("❌ Error fetching campaigns:", error);
    return res.status(500).json({ message: "Internal Server Error" });
  }
};

module.exports = {
  show,
  showAll,
};
