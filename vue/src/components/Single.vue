<template>
  <div>
    <div class="run-header">
      <div class="run-id-container">
        <span class="run-label">Run ID:</span>
        <span class="run-id">{{ $route.query.runId }}</span>
      </div>
      <div v-if="input.bucketiseSeconds" class="run-details">
        Re-bucketised into {{ input.bucketiseSeconds }} second buckets, merged with {{ input.mergingType }}.
        <b-alert variant="danger" show>
          Currently if re-bucketising data, metrics are disabled as re-bucketising the
          JSON-based metrics is non-trivial.
        </b-alert>
      </div>
    </div>

    <div v-if="loading" class="text-center">
      <b-spinner variant="primary"></b-spinner>
      <p>Fetching...</p>
    </div>

    <Results v-else :single="input" :input="input" :showDropdown="false"></Results>
  </div>
</template>

<script>
import Results from "@/components/Results.vue";

export default {
  components: {Results},
  data() {
    return {
      loading: true,
      input: {
        yAxes: [
          {
            type: "buckets",
            yAxisID: "left",
            column: "operations_total",
          },
          {
            type: "buckets",
            yAxisID: "right",
            column: "duration_average_us",
          },
           // Too slow to fetch all these
          // {
          //   type: "buckets",
          //   yAxisID: "right",
          //   column: "duration_p99_us",
          // },
          // {
          //   type: "buckets",
          //   yAxisID: "right",
          //   column: "duration_max_us",
          // },  
          {
            type: "errors",
            yAxisID: "left",
          }
        ],
        annotations: [{type: "run-events"}],
        runId: this.$route.query.runId,
        trimmingSeconds: 0,
        mergingType: this.$route.query.mergingType ?? "Average",
        bucketiseSeconds: this.$route.query.bucketiseSeconds ?? 10,
      }
    }
  },
  async created() {
    await this.initializeWithMetrics();
  },
  methods: {
    async initializeWithMetrics() {
      try {
        const response = await fetch(`${document.location.protocol}//${document.location.hostname}:3002/dashboard/metrics`);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const metrics = await response.json();
        
        if (Array.isArray(metrics)) {
          // Add all metrics to yAxes with display: false by default
          metrics.forEach(metric => {
            this.input.yAxes.push({
              type: "metric",
              yAxisID: "right",
              metric: metric,
              
            });
          });
        }
      } catch (error) {
        console.error('Error fetching metrics:', error);
      } finally {
        this.loading = false;
      }
    }
  }
}
</script>

<style scoped>
.run-header {
  background: #ffffff;
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.run-id-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.run-label {
  font-size: 1.125rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.run-id {
  font-family: 'Monaco', 'Menlo', 'Courier New', monospace;
  font-size: 1.25rem;
  color: #2563eb;
  font-weight: 600;
  padding: 0.5rem 1rem;
  background: #f1f5f9;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.run-details {
  color: var(--text-secondary);
  font-size: 0.875rem;
}
</style>
