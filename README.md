# Movie-app
Code repository for NIIT Capstone project - Movie App

@Override
    public HashMap<String, Object> createPmsRedemption(RedemptionsRequestDTO requestDTO, String type,
                                                       String panNumber, String investerPanNo, String initiator, String portal) {
        logger.info("Entering createRedemptionTransaction API  Service Imple | requestDTO: " + requestDTO + " | type: " + type + " | panNumber: " + panNumber + " | investerPanNo: " + investerPanNo + " | initiator: " + initiator + " | portal: " + portal);
        AdditionalPurchaseResponseDTO response = new AdditionalPurchaseResponseDTO();
        HashMap<String, Object> hmResponse = new HashMap<>();
        String InvestorPanNumber = "";

        if (type.equalsIgnoreCase("PMS") && null == requestDTO.getReferenceSystematicTransactionNo()) {
            if (initiator.equalsIgnoreCase(MessageConstant.DISTRIBUTOR_KEY)) {
                InvestorPanNumber = investerPanNo;
            } else {
                InvestorPanNumber = panNumber;
            }
            List<ClientCodeRequestDTO> clientcodes = commonTransactionUtils.getClientCodesByPanNumber(InvestorPanNumber);
            if (!CommonUtils.isNullOrEmpty(clientcodes)) {
                Double totalEnteredAmount = requestDTO.getRedemptions().stream().mapToDouble(data -> Double.parseDouble(data.getRedemptionAmount())).sum();
                logger.info("createPmsRedemption() service impl |  totalEnteredAmount " + totalEnteredAmount);
                Double panLevelTotalAum = commonTransactionUtils.getTotalAumAmountByclientCodes(clientcodes);
                logger.info("createPmsRedemption() service impl |  panLevelTotalAum " + panLevelTotalAum);
                if (!CommonUtils.isNullOrEmpty(panLevelTotalAum) && (panLevelTotalAum - totalEnteredAmount) > 0 && (panLevelTotalAum - totalEnteredAmount) < MessageConstant.PMS_REDEMPTION_FIFTYTWO_LACKS_AMOUNT) {
                    response.setStatus("failed");
                    response.setMessage("AUM of PAN combination less than 52 Lakhs. Please contact your RM. ");
                    hmResponse.put("RedemptionResponse", response);
                    logger.info("createRedemptionTransaction API  Service Imple | failed | AUM of PAN combination less than 52 Lakhs. Please contact your RM.  ");
                    return hmResponse;
                }
            }

            for (RedemptionRequestDTO redemptionTransaction : requestDTO.getRedemptions()) {

                if (redemptionTransaction.getRedemptionType().equalsIgnoreCase("Partial Redemption") && Double.parseDouble(redemptionTransaction.getRedemptionAmount()) < MessageConstant.PMS_REDEMPTION_TWO_LACKS) {
                    response.setStatus("failed");
                    response.setMessage("Entered Amount for Pms Redemption on " + redemptionTransaction.getClientCode() + " cannot be less than 2Lacks");
                    logger.info("createRedemptionTransaction API  Service Imple | failed | Entered Amount for Pms Redemption on " + redemptionTransaction.getClientCode() + " cannot be less than 2Lacks");
                    hmResponse.put("RedemptionResponse", response);
                    return hmResponse;
                }
                double pendingRedemptionAmount = transactionMasterRepository.getTotalPendingRedemptionAmount(redemptionTransaction.getClientCode());
                TotalPmsAumAmountWidgetDTO pmsAumData = new TotalPmsAumAmountWidgetDTO();
                pmsAumData = clientAumCashRepository.getTotalPmsAumAmount(redemptionTransaction.getClientCode());
                if (!CommonUtils.isNullOrEmpty(pmsAumData.getPmsAmount())) {
                    if (redemptionTransaction.getRedemptionType().equalsIgnoreCase("Partial Redemption") && (pmsAumData.getPmsAmount() - pendingRedemptionAmount - Double.parseDouble(redemptionTransaction.getRedemptionAmount())) < MessageConstant.PMS_REDEMPTION_FIFTEEN_LACK) {
                        response.setStatus("failed");
                        response.setMessage("For Pms Redemption on " + redemptionTransaction.getClientCode() + ", the AUM cannot be less than 15 Lakhs. Please enter the amount again.");
                        hmResponse.put("RedemptionResponse", response);
                        logger.info("createRedemptionTransaction API  Service Imple | failed | For Pms Redemption on " + redemptionTransaction.getClientCode() + ", the AUM cannot be less than 15 Lakhs. Please enter the amount again.");
                        return hmResponse;
                    }
                }

            }
        }

        for (RedemptionRequestDTO redemptionTransaction : requestDTO.getRedemptions()) {
            TransactionMasterDTO transactionMasterDTO = transactionMasterRepository.getFullRedemptionTransaction(redemptionTransaction.getClientCode());
            String chargeUpto = clientMasterRepository.getChargeUptoDateByClientCode(redemptionTransaction.getClientCode());
            if (null != transactionMasterDTO || null != chargeUpto) {
                response.setStatus("failed");
                response.setMessage("Full Redemption is placed for " + redemptionTransaction.getClientCode() + " Kindly contact your relationship manager for assistance.");
                hmResponse.put("RedemptionResponse", response);
                logger.info("Full Redemption is placed for " + redemptionTransaction.getClientCode() + ", Kindly contact your relationship manager for assistance.");
                return hmResponse;
            }
        }

        response.setStatus("success");
        requestDTO.getRedemptions().forEach(redemptionTransaction -> {
            String panNo = "";
            TransactionTypeMasterDTO transactiontypeDTO = null;
            StrategyTypeMasterDTO strategyTypeDTO = null;

            // Transaction creation
            transactiontypeDTO = transactionTypeRepository.getTransactionTypeMaster(6); // transaction Type 6 = Cash

            logger.info("transactiontypeDTO " + transactiontypeDTO);
            String transactionType = transactiontypeDTO.getDescription();
            String transactionNumber = "";
            if (portal.equalsIgnoreCase("Niveus Portal")) {
                transactionNumber = CommonUtils.generateTransactionNumber(type, transactionType);
            } else if (portal.equalsIgnoreCase("Deal")) {
                transactionNumber = CommonUtils.generateDLWCTransactionNumber("DL", type, transactionType);
            } else {
                transactionNumber = CommonUtils.generateDLWCTransactionNumber("WC", type, transactionType);
            }

            strategyTypeDTO = strategyTypeRepository.getStrategyType(type);

            int strategyTypeId = Integer.parseInt(strategyTypeDTO.getId());
            logger.info("Fetched getStrategyType " + strategyTypeId);
            int redemptionTypeId = redemptionTypeRepository
                    .getRedemptionTypeId(redemptionTransaction.getRedemptionType());
            logger.info("Fetched getRedemptionType " + redemptionTypeId);
            int transferTypeId = transferTypeRespository.getTransferTypeId(redemptionTransaction.getTransferType());
            logger.info("Fetched getTransferTypeId " + transferTypeId);
            TransactionMasterDTO transactionMaster = new TransactionMasterDTO();
            transactionMaster.setTransactionNo(transactionNumber);
            transactionMaster.setTransactionTypeId(6);
            transactionMaster.setStrategyTypeId(strategyTypeId);
            transactionMaster.setLevel(MessageConstant.LEVEL_1);
            transactionMaster.setModule(MessageConstant.REDEMPTION_MODULE);
            transactionMaster.setClientCode(redemptionTransaction.getClientCode());
            if (initiator.equalsIgnoreCase(MessageConstant.INVESTOR_KEY)) {
                // setting invester panno from jwt token
                transactionMaster.setInitiator(panNumber);
                panNo = panNumber;
            } else {
                // setting selected invester panno
                transactionMaster.setInitiator(investerPanNo);
                // setting distributor panno
                transactionMaster.setDistributorPanNo(panNumber);
                panNo = investerPanNo;
            }
            transactionMaster.setInitiatorRole(initiator);
            transactionMaster.setTransactionAmount(redemptionTransaction.getRedemptionAmount());
            transactionMaster.setRedemptionTypeId(redemptionTypeId);
            transactionMaster.setTransferTypeId(transferTypeId);
            transactionMaster.setExcludeDeduction(redemptionTransaction.getExcludeDeduction());
            transactionMaster.setJhApprovalCount(0);
            transactionMaster.setTransactionCategory(requestDTO.getTransactionCategory());
            transactionMaster.setReferenceSystematicTransactionNo(requestDTO.getReferenceSystematicTransactionNo());

            if (!portal.equalsIgnoreCase("Niveus Portal")) {
                transactionMaster.setConsolidatedRedemptionLogId(Integer.parseInt(redemptionTransaction.getConsolidatedId()));
                transactionMaster.setConsolidatedSourceTransactionId(redemptionTransaction.getWCDLId());
            }

            // 1 is passed to insert record into transaction master
            if (CommonUtils.isNullOrEmpty(requestDTO.getReferenceSystematicTransactionNo())) {
                commonTransactionUtils.insertOrUpdateTransaction(transactionMaster, null, 1);
            }
            logger.info("transactionMasterDetails " + transactionMaster);
            hmResponse.put("transactionMasterDetails", transactionMaster);


            if (null == requestDTO.getReferenceSystematicTransactionNo() && initiator.equalsIgnoreCase(MessageConstant.INVESTOR_KEY)) {
                this.redemptionSendEmail1(redemptionTransaction.getClientCode(), transactionNumber, initiator,
                        transferTypeId, panNo, type);
            } else {
                if (null == requestDTO.getReferenceSystematicTransactionNo()) {
                    InvestorDetailsWidgetDTO investorDetails = clientMasterRepository
                            .getInvestorDetails(requestDTO.getDistributorInvesterPanNo());
                    logger.info("Fetched getInvestorDetails " + investorDetails);
                    // Send mail to Invester
                    String subject = String.format(EmailTemplateConstant.MAIL_TEMPLATE_SUBJECT_INVESTER_REDEMPTION_REQUEST,
                            redemptionTransaction.getSchemeName(), redemptionTransaction.getClientCode());
                    String content = String.format(EmailTemplateConstant.MAIL_TEMPLATE_CONTENT_INVESTER_REDEMPTION_REQUEST,
                            investorDetails.getClientName(), redemptionTransaction.getRedemptionAmount(),
                            redemptionTransaction.getSchemeName(), redemptionTransaction.getClientCode());

                    SendEmailNameDTO fromDTO = new SendEmailNameDTO();
                    fromDTO.setEmail(fromEmailId);
                    fromDTO.setName(fromEmailName);

                    commonTransactionUtils.sendEmail(redemptionTransaction.getEmail(), subject, content, null,
                            emailAuthorizationKey, commonEmailServerUrl, fromDTO);
                }

                // change status to 1(Request submitted)
                TransactionMasterDTO transactionStatusUpdate = new TransactionMasterDTO();
                if (CommonUtils.isNullOrEmpty(requestDTO.getReferenceSystematicTransactionNo())) {
                    transactionStatusUpdate.setLevel(MessageConstant.LEVEL_2);
                } else {
                    transactionStatusUpdate.setLevel(MessageConstant.LEVEL_1);
                }
                transactionStatusUpdate.setModule(MessageConstant.REDEMPTION_MODULE);
                transactionStatusUpdate.setTransactionNo(transactionNumber);
                transactionStatusUpdate.setClientCode(redemptionTransaction.getClientCode());
                transactionStatusUpdate.setInitiatorRole(initiator);
                transactionStatusUpdate.setRedemptionTypeId(redemptionTypeId);
                transactionStatusUpdate.setTransferTypeId(transferTypeId);
                // 2 is passed to update Transaction Master
                if (CommonUtils.isNullOrEmpty(requestDTO.getReferenceSystematicTransactionNo())) {
                    commonTransactionUtils.insertOrUpdateTransaction(transactionStatusUpdate, null, 2);
                }

            }

            response.setStatus(transactionMasterRepository.getTransactionStatus(transactionNumber));
            response.setTransactionNo(transactionNumber);
            hmResponse.put("RedemptionResponse", response);

            if (initiator.equalsIgnoreCase(MessageConstant.DISTRIBUTOR_KEY) && !requestDTO.getTransactionCategory().contains("Switch")) {
                if(type.equalsIgnoreCase(MessageConstant.PMS)){
                    approveTransactionUtils.sendDistributorLink(requestDTO.getRedemptions().get(0).getRedemptionType(), transactionNumber, transactionMaster.getClientCode(), MessageConstant.TXN_TYPE_REDEMPTION);
                } else if (type.equalsIgnoreCase(MessageConstant.AIF)) {
                    approveTransactionUtils.sendDistributorLink(requestDTO.getRedemptions().get(0).getRedemptionType(), transactionNumber, transactionMaster.getClientCode(), MessageConstant.TXN_TYPE_AIF_REDEMPTION);
                }
            }
        });

        logger.info("Exiting createRedemptionTransaction API" + (HashMap<String, Object>) hmResponse);
        return (HashMap<String, Object>) hmResponse;
    }
