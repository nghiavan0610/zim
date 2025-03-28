# AWS Cost Optimization

## Task Overview

The AWS bill has increased due to several factors, primarily related to data transfer costs, EC2 instance utilization, and S3 storage choices.

## Main Causes of Increased Costs

1. **High Data Transfer Costs for Video Streaming**: The costs associated with transferring large amounts of video data can significantly impact the overall AWS bill.
2. **Over-Provisioned EC2 Instances**: Instances that are underutilized during non-peak hours lead to unnecessary expenses.
3. **S3 Standard Storage for Videos**: Storing videos, including archived content that is rarely accessed, in S3 Standard storage incurs higher costs than necessary.

## Actions to Reduce AWS Costs

1. **Implement Data Transfer Optimization**:

   - Use Amazon CloudFront or Cloudflare or Bunny as a CDN to cache video content closer to users, reducing data transfer costs.
   - **Impact**: This will lower data transfer fees and improve user experience by decreasing latency.

2. **Right-Size EC2 Instances**:

   - Analyze EC2 instance usage and resize or schedule instances to match demand, using AWS Auto Scaling to adjust capacity based on traffic.
   - **Impact**: This will reduce costs by ensuring that only the necessary resources are provisioned, especially during non-peak hours.

3. **Transition to S3 Intelligent-Tiering**:

   - Move infrequently accessed videos to S3 Intelligent-Tiering, which automatically moves data between two access tiers when access patterns change.
   - **Impact**: This will lower storage costs while maintaining performance, as frequently accessed data remains in the standard tier.

4. **Utilize R2 for Video Storage**:
   - Transition video storage to Cloudflare R2, which offers cost-effective storage and free egress to Cloudflare's network, significantly reducing data transfer costs.
   - **Impact**: This will lower overall storage and transfer expenses while improving performance through faster content delivery and reduced latency.
